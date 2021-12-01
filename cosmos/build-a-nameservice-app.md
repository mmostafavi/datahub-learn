
[**The original tutorial can be found in the Cosmos SDK documentation here**](https://tutorials.cosmos.network/nameservice/tutorial/00-intro.html#). 

# Introduction

In this tutorial, you will build a functional [Cosmos SDK](https://github.com/cosmos/cosmos-sdk/) application and, in the process, learn the basic concepts and structures of the SDK. The example will showcase how quickly and easily you can **build your own blockchain from scratch** on top of the Cosmos SDK.

By the end of this tutorial you will have a functional `nameservice` application, a mapping of strings to other strings \(`map[string]string`\). This is similar to [Namecoin](https://namecoin.org/), [ENS](https://ens.domains/), or [Handshake](https://handshake.org/), which all model the traditional DNS systems \(`map[domain]zonefile`\). Users will be able to buy unused names, or sell/trade their name.

All of the final source code for this tutorial project is in this directory \(and compiles\). However, it is best to follow along manually and try building the project yourself!

# Requirements

* [`golang` &gt;1.13.0](https://golang.org/doc/install) installed
* Github account and either [Github CLI](https://hub.github.com/) or [Github Desktop \(64-bit required\)](https://help.github.com/en/desktop/getting-started-with-github-desktop/installing-github-desktop)
* Desire to create your own blockchain!
* The [scaffold tool](https://github.com/cosmos/scaffold) will be used to go through this tutorial. Clone the repo and install with `git clone git@github.com:cosmos/scaffold.git && cd scaffold && make`. Check out the repo for more detailed instructions.

# Getting Started

Through the course of this tutorial you will create the following files that make up your application:

```text
./nameservice
├── Makefile
├── Makefile.ledger
├── app.go
├── cmd
│   ├── nscli
│   │   └── main.go
│   └── nsd
│       └── main.go
├── go.mod
├── go.sum
└── x
    └── nameservice
        ├── alias.go
        ├── client
        │   ├── cli
        │   │   ├── query.go
        │   │   └── tx.go
        │   └── rest
        │       ├── query.go
        │       ├── rest.go
        │       └── tx.go
        ├── genesis.go
        ├── handler.go
        ├── keeper
        │   ├── keeper.go
        │   └── querier.go
        ├── types
        │   ├── codec.go
        │   ├── errors.go
        │   ├── expected_keepers.go
        │   ├── key.go
        │   ├── msgs.go
        │   ├── querier.go
        │   └── types.go
        └── module.go
```

Follow along! The first step describes the design of your application.

# Application Goals

The goal of the application you are building is to let users buy names and to set a value these names resolve to. The owner of a given name will be the current highest bidder. In this section, you will learn how these simple requirements translate to application design.

A blockchain application is just a [replicated deterministic state machine](https://en.wikipedia.org/wiki/State_machine_replication). As a developer, you just have to define the state machine \(i.e. what the state, a starting state and messages that trigger state transitions\), and [_Tendermint_](https://docs.tendermint.com/master/introduction/) will handle replication over the network for you.

> Tendermint is an application-agnostic engine that is responsible for handling the _networking_ and _consensus_ layers of your blockchain. In practice, this means that Tendermint is responsible for propagating and ordering transaction bytes. Tendermint Core relies on an eponymous Byzantine-Fault-Tolerant \(BFT\) algorithm to reach consensus on the order of transactions. For more on Tendermint, click [here](https://tendermint.com/docs/introduction/introduction.html).

The [Cosmos SDK](https://github.com/cosmos/cosmos-sdk/) is designed to help you build state machines. The SDK is a **modular framework**, meaning applications are built by aggregating a collection of interoperable modules. Each module contains its own message/transaction processor, while the SDK is responsible for routing each message to its respective module.

Here are the modules you will need for the nameservice application:

* `auth`: This module defines accounts and fees and gives access to these functionalities to the rest of your application.
* `bank`: This module enables the application to create and manage tokens and token balances.
* `staking` : This module enables the application to have validators that people can delegate to.
* `distribution` : This module give a functional way to passively distribute rewards between validators and delegators.
* `slashing` : This module disincentivizes people with value staked in the network, ie. Validators.
* `supply` : This module holds the total supply of the chain.
* `nameservice`: This module does not exist yet! It will handle the core logic for the `nameservice` application you are building. It is the main piece of software you have to work on to build your application.

Now, take a look at the two main parts of your application: the **state** and the message **types**.

# State

The state represents your application at a given moment. It tells how much token each account possesses, what are the owners and price of each name, and to what value each name resolves to.

The state of tokens and accounts is defined by the `auth` and `bank` modules, which means you don't have to concern yourself with it for now. What you need to do is define the part of the state that relates specifically to your `nameservice` module.

In the SDK, everything is stored in one store called the `multistore`. Any number of key/value stores \(called [`KVStores`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#KVStore) in the Cosmos SDK\) can be created in this multistore. For this application, we will use one store to map `name`s to its respective `whois`, a struct that holds a name's value, owner, and price.

# Messages

Messages are contained in transactions. They trigger state transitions. Each module defines a list of messages and how to handle them. Here are the messages you need to implement the desired functionality for your nameservice application:

* `MsgSetName`: This message allows name owners to set a value for a given name.
* `MsgBuyName`: This message allows accounts to buy a name and become its owner.
  * When someone buys a name, they are required to pay the previous owner of the name a price higher than the price the previous owner paid for it. If a name does not have a previous owner yet, they must burn a `MinPrice` amount.

When a transaction \(included in a block\) reaches a Tendermint node, it is passed to the application via the [ABCI](https://github.com/tendermint/tendermint/tree/master/abci) and decoded to get the message. The message is then routed to the appropriate module and handled there according to the logic defined in the `Handler`. If the state needs to be updated, the `Handler` calls the `Keeper` to perform the update. You will learn more about these concepts in the next steps of this tutorial.

Now that you have decided on how your application functions from a high-level perspective, it is time to start implementing it.

# Create your application

To create a new app, we will be using the lvl-1 app which is provided by the scaffold tool.

You can fill in `user` with your Github username and `repo` with the name of the repo you are creating.

```text
scaffold app lvl-1 [user] [repo] [flags]

cd [repo]
```

In `app.go` it is defined what the application does when it receives a transaction. But first, it needs to be able to receive transactions in the correct order. This is the role of the [Tendermint consensus engine](https://github.com/tendermint/tendermint).

Links to godocs for each module and package imported:

* [`log`](https://godoc.org/github.com/tendermint/tendermint/libs/log): Tendermint's logger.
* [`auth`](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth): The `auth` module for the Cosmos SDK.
* [`dbm`](https://godoc.org/github.com/tendermint/tm-db): Code for working with the Tendermint database.
* [`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp): See below

A couple of the packages here are `tendermint` packages. Tendermint passes transactions from the network to the application through an interface called the [ABCI](https://docs.tendermint.com/master/spec/abci/). If you look at the architecture of the blockchain node you are building, it looks like the following:

```text
+---------------------+
|                     |
|     Application     |
|                     |
+--------+---+--------+
         ^   |
         |   | ABCI
         |   v
+--------+---+--------+
|                     |
|                     |
|     Tendermint      |
|                     |
|                     |
+---------------------+
```

Fortunately, you do not have to implement the ABCI interface. The Cosmos SDK provides a boilerplate implementation of it in the form of [`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp).

Here is what `baseapp` does:

* Decode transactions received from the Tendermint consensus engine.
* Extract messages from transactions and do basic sanity checks.
* Route the message to the appropriate module so that it can be processed. Note that `baseapp` has no knowledge of the specific modules you want to use. It is your job to declare such modules in `app.go`, as you will see later in this tutorial. `baseapp` only implements the core routing logic that can be applied to any module.
* Commit if the ABCI message is [`DeliverTx`](https://docs.tendermint.com/master/spec/abci/abci.html#delivertx) \([`CheckTx`](https://docs.tendermint.com/master/spec/abci/abci.html#checktx) changes are not persistent\).
* Help set up [`Beginblock`](https://docs.tendermint.com/master/spec/abci/abci.html#beginblock) and [`Endblock`](https://docs.tendermint.com/master/spec/abci/abci.html#endblock), two messages that enable you to define logic executed at the beginning and end of each block. In practice, each module implements its own `BeginBlock` and `EndBlock` sub-logic, and the role of the app is to aggregate everything together \(_Note: you won't be using these messages in your application_\).
* Help initialize your state.
* Help set up queries.

Now you need to rename the `appName` & `NewApp` types to the name of your app. In this case you can use `nameservice` & `NameServiceApp`. This type will embed `baseapp` \(embedding in Go similar to inheritance in other languages\), meaning it will have access to all of `baseapp`'s methods.

Great! You now have the start of your application. Currently you have a working blockchain, but we will customize it throughout this tutorial.

`baseapp` has no knowledge of the routes or user interactions you want to use in your application. The primary role of your application is to define these routes. Another role is to define the initial state. Both these things require that you add modules to your application.

As you have seen in the [application design](https://tutorials.cosmos.network/nameservice/tutorial/app-design.html) section, you need a few modules for your nameservice: `auth`, `bank`, `staking`, `distribution`, `slashing` and `nameservice`. The first five already exist, but not the last! The `nameservice` module will define the bulk of your state machine. The next step is to build it.

In order to complete your application, you need to include modules. Go ahead and start building your nameservice module. You will come back to `app.go` later. 

# Types 

First thing we're going to do is create a module in the `/x/` folder with the scaffold tool using the below command:

In the case of this tutorial we will be naming the module `nameservice`

```
cd x/

scaffold module [user] [repo] nameservice
```

**`types.go`**

Now we can continue with creating a module. Start by creating the file `types.go`in `./x/nameservice/types` folder which will hold customs types for the module.

## Whois

Each name will have three pieces of data associated with it.

* Value - The value that a name resolves to. This is just an arbitrary string, but in the future you can modify this to require it fitting a specific format, such as an IP address, DNS Zone file, or blockchain address.
* Owner - The address of the current owner of the name
* Price - The price you will need to pay in order to buy the name

To start your SDK module, define your `nameservice.Whois` struct in the `./x/nameservice/types/types.go` file:

```javascript
package types

import (
	"fmt"
	"strings"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

// MinNamePrice is Initial Starting Price for a name that was never previously owned
var MinNamePrice = sdk.Coins{sdk.NewInt64Coin("nametoken", 1)}

// Whois is a struct that contains all the metadata of a name
type Whois struct {
	Value string         `json:"value"`
	Owner sdk.AccAddress `json:"owner"`
	Price sdk.Coins      `json:"price"`
}

// NewWhois returns a new Whois with the minprice as the price
func NewWhois() Whois {
	return Whois{
		Price: MinNamePrice,
	}
}

// implement fmt.Stringer
func (w Whois) String() string {
	return strings.TrimSpace(fmt.Sprintf(`Owner: %s
Value: %s
Price: %s`, w.Owner, w.Value, w.Price))
}
```

As mentioned in the [Design doc](https://tutorials.cosmos.network/nameservice/tutorial/app-design.html), if a name does not already have an owner, we want to initialize it with some MinPrice.

## Key

Start by navigating to the `key.go` file in the types folder. Within the `key.go` file, you will see that the keys which will be used throughout the creation of the module have already been created.

Defining keys that will be used throughout the application helps with writing [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) code.

```go
package types

const (
	// ModuleName is the name of the module
	ModuleName = "nameservice"

	// StoreKey to be used when creating the KVStore
	StoreKey = ModuleName

	// RouterKey is the module name router key
	RouterKey = ModuleName

	// QuerierRoute to be used for querierer msgs
	QuerierRoute = ModuleName
)
```

## Errors

Start by navagating to the `errors.go` file within the types folder. Within your `errors.go` file, define errors that are custom to your module along with their codes.

```go
package types

import (
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

var (
	ErrNameDoesNotExist = sdkerrors.Register(ModuleName, 1, "name does not exist")
)
```

You must also add the corresponding method that'll be called at the time of error handling. For instance, let's say we try to delete a name that is not present in the store. In this case, an error should be thrown as the name does not exist.

We'll see later on in the tutorial where this method is called.

Now we move on to writing the Keeper for the module.

## Expected Keepers

Next create a file within the `/types` directory called `expected_keepers.go`. In this file we will be defining what we expect other modules to have.

For example in the nameservice module we will be using the bank module to facilitate transfers between two parties. To make this happen we will rely on two functions from the bank module.

* `SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)`



* `SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error`

you can see below how the file is structured below:

```go
package types

import (
	sdk "github.com/cosmos/cosmos-sdk/types"
)

// When a module wishes to interact with an otehr module it is good practice to define what it will use
// as an interface so the module can not use things that are not permitted.
type BankKeeper interface {
	SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)
	SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error
}
```

# The Keeper

The main core of a Cosmos SDK module is a piece called the `Keeper`. It is what handles interaction with the store, has references to other keepers for cross-module interactions, and contains most of the core functionality of a module.

## Keeper Struct

To start your SDK module, define your `nameservice.Keeper` in the `./x/nameservice/keeper/keeper.go` file. Defined in this generated file are a few extra items that we will not cover at this time, for this reason we will start by clearing the `keeper.go` file in favor of following this tutorial.

```go
package keeper

import (
	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
)

// Keeper maintains the link to data storage and exposes getter/setter methods for the various parts of the state machine
type Keeper struct {
	CoinKeeper types.BankKeeper

	storeKey  sdk.StoreKey // Unexposed key to access store from sdk.Context

	cdc *codec.Codec // The wire codec for binary encoding/decoding.
}

```

A couple of notes about the above code:

* Two `cosmos-sdk` packages and `types` for your application are imported:
  * [`codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec) - the `codec` provides tools to work with the Cosmos encoding format, [Amino](https://github.com/tendermint/go-amino).
  * [`types` \(as sdk\)](https://godoc.org/github.com/cosmos/cosmos-sdk/types) - this contains commonly used types throughout the SDK.
  * `types` - it contains `BankKeeper` you have defined in previous section. 
* The `Keeper` struct. In this keeper there are a couple of key pieces:
  * `types.BankKeeper` - This is an interface you had defined previous section to use `bank` module. Including it allows code in this module to call functions from the `bank` module. The SDK uses an [object capabilities](https://en.wikipedia.org/wiki/Object-capability_model) approach to accessing sections of the application state. This is to allow developers to employ a least authority approach, limiting the capabilities of a faulty or malicious module from affecting parts of state it doesn't need access to.
  * [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec) - This is a pointer to the codec that is used by Amino to encode and decode binary structs.
  * [`sdk.StoreKey`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#StoreKey) - This is a store key which gates access to a `sdk.KVStore` which persists the state of your application: the Whois struct that the name points to \(i.e. `map[name]Whois`\).

## Getters and Setters

Now it is time to add methods to interact with the stores through the `Keeper`. First, add a function to set the Whois a given name resolves to:

```go
// Sets the entire Whois metadata struct for a name
func (k Keeper) SetWhois(ctx sdk.Context, name string, whois types.Whois) {
	if whois.Owner.Empty() {
		return
	}
	store := ctx.KVStore(k.storeKey)
	store.Set([]byte(name), k.cdc.MustMarshalBinaryBare(whois))
}
```

In this method, first get the store object for the `map[name]Whois` using the the `storeKey` from the `Keeper`.

> _NOTE_: This function uses the [`sdk.Context`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Context). This object holds functions to access a number of important pieces of the state like `blockHeight` and `chainID`.

Next, you insert the `<name, whois>` pair into the store using its `.Set([]byte, []byte)` method. As the store only takes `[]byte`, we use the Cosmos SDK encoding library called Amino to marshal the `Whois` struct to `[]byte` to be inserted into the store.

If the owner field of a `Whois` is empty, we do not write anything to the store, as all names that exist must have an owner.

Next, add a method to resolve the names \(i.e. look up the `Whois` for the `name`\):

```go
// Gets the entire Whois metadata struct for a name
func (k Keeper) GetWhois(ctx sdk.Context, name string) types.Whois {
	store := ctx.KVStore(k.storeKey)
	if !k.IsNamePresent(ctx, name) {
		return types.NewWhois()
	}
	bz := store.Get([]byte(name))
	var whois types.Whois
	k.cdc.MustUnmarshalBinaryBare(bz, &whois)
	return whois
}

```

Here, like in the `SetName` method, first access the store using the `storeKey`. Next, instead of using the `Set` method on the store key, use the `.Get([]byte) []byte` method. As the parameter into the function, pass the key, which is the `name` string casted to `[]byte`, and get back the result in the form of `[]byte`. We once again use Amino, but this time to unmarshal the byteslice back into a `Whois` struct which we then return.

If a name currently does not exist in the store, it returns a new `Whois`, which has the minimumPrice initialized in it.

Next, add a method to delete the names:

```go
// Deletes the entire Whois metadata struct for a name
func (k Keeper) DeleteWhois(ctx sdk.Context, name string) {
	store := ctx.KVStore(k.storeKey)
	store.Delete([]byte(name))
}
```

Here, like in the `SetName` method, first access the store using the `storeKey`. Next, we delete the name from the store using its `.Delete([]byte)` method. As the store only takes `[]byte`, the `name` string is casted to `[]byte` while passing it as a function parameter.

Now, we add functions for getting specific parameters from the store based on the name. However, instead of rewriting the store getters and setters, we reuse the `GetWhois` and `SetWhois` functions. For example, to set a field, first we grab the whole `Whois` data, update our specific field, and put the new version back into the store.

```go
// ResolveName - returns the string that the name resolves to
func (k Keeper) ResolveName(ctx sdk.Context, name string) string {
	return k.GetWhois(ctx, name).Value
}

// SetName - sets the value string that a name resolves to
func (k Keeper) SetName(ctx sdk.Context, name string, value string) {
	whois := k.GetWhois(ctx, name)
	whois.Value = value
	k.SetWhois(ctx, name, whois)
}

// HasOwner - returns whether or not the name already has an owner
func (k Keeper) HasOwner(ctx sdk.Context, name string) bool {
	return !k.GetWhois(ctx, name).Owner.Empty()
}

// GetOwner - gets the current owner of a name
func (k Keeper) GetOwner(ctx sdk.Context, name string) sdk.AccAddress {
	return k.GetWhois(ctx, name).Owner
}

// SetOwner - sets the current owner of a name
func (k Keeper) SetOwner(ctx sdk.Context, name string, owner sdk.AccAddress) {
	whois := k.GetWhois(ctx, name)
	whois.Owner = owner
	k.SetWhois(ctx, name, whois)
}

// GetPrice - gets the current price of a name
func (k Keeper) GetPrice(ctx sdk.Context, name string) sdk.Coins {
	return k.GetWhois(ctx, name).Price
}

// SetPrice - sets the current price of a name
func (k Keeper) SetPrice(ctx sdk.Context, name string, price sdk.Coins) {
	whois := k.GetWhois(ctx, name)
	whois.Price = price
	k.SetWhois(ctx, name, whois)
}

// Check if the name is present in the store or not
func (k Keeper) IsNamePresent(ctx sdk.Context, name string) bool {
	store := ctx.KVStore(k.storeKey)
	return store.Has([]byte(name))
}
```

The SDK also includes a feature called an `sdk.Iterator`, which returns an iterator over all the `<Key, Value>` pairs in a specific spot in a store. We will add a function to get an iterator over all the names that exist in the store.

```go
// Get an iterator over all names in which the keys are the names and the values are the whois
func (k Keeper) GetNamesIterator(ctx sdk.Context) sdk.Iterator {
	store := ctx.KVStore(k.storeKey)
	return sdk.KVStorePrefixIterator(store, []byte{})
}
```

The last piece of code needed in the `./x/nameservice/keeper/keeper.go` file is a constructor function for `Keeper`:

```go
// NewKeeper creates new instances of the nameservice Keeper
func NewKeeper(cdc *codec.Codec, storeKey sdk.StoreKey, coinKeeper types.BankKeeper) Keeper {
	return Keeper{
		cdc:        cdc,
		storeKey:   storeKey,
		CoinKeeper: coinKeeper,
	}
}
```

Next, its time to move onto describing how users interact with your new store using `Msgs` and `Handlers`.

# Msgs and Handlers 

Now that you have the `Keeper` setup, it is time to build the `Msgs` and `Handlers` that actually allow users to buy names and set values for them.

**`Msgs`**

`Msgs` trigger state transitions. `Msgs` are wrapped in [`Txs`](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L41) that clients submit to the network. The Cosmos SDK wraps and unwraps `Msgs` from `Txs`, which means, as an app developer, you only have to define `Msgs`. `Msgs` must satisfy the following interface \(we'll implement all of these in the next section\):

```go
// Transactions messages must fulfill the Msg
type Msg interface {
	// Return the message type.
	// Must be alphanumeric or empty.
	Type() string

	// Returns a human-readable string for the message, intended for utilization
	// within tags
	Route() string

	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() Error

	// Get the canonical byte representation of the Msg.
	GetSignBytes() []byte

	// Signers returns the addrs of signers that must sign.
	// CONTRACT: All signatures must be present to be valid.
	// CONTRACT: Returns addrs in some deterministic order.
	GetSigners() []AccAddress
}


	// Transactions messages must fulfill the Msg type 
	Msg interface { 
	// Return the message type. 
	// Must be alphanumeric or empty. 
	Type() string 
	// Returns a human-readable string for the message, intended for utilization
	// within tags 
	Route() string 
	// ValidateBasic does a simple validation check that 
	// doesn't require access to any other information. 
	ValidateBasic() Error 
	// Get the canonical byte representation of the Msg. 
	GetSignBytes() []byte 
	// Signers returns the addrs of signers that must sign. 
	// CONTRACT: All signatures must be present to be valid. 
	// CONTRACT: Returns addrs in some deterministic order. 
	GetSigners() []AccAddress 
}
```

**`Handlers`**

`Handlers` define the action that needs to be taken \(which stores need to get updated, how, and under what conditions\) when a given `Msg` is received.

In this module you have three types of `Msgs` that users can send to interact with the application state: [`SetName`](https://tutorials.cosmos.network/nameservice/tutorial/set-name.html), [`BuyName`](https://tutorials.cosmos.network/nameservice/tutorial/buy-name.html) and [`DeleteName`](https://tutorials.cosmos.network/nameservice/tutorial/delete-name.html). They will each have an associated `Handler`.

Now that you have a better understanding of `Msgs` and `Handlers`, you can start building your first message: `SetName`

## SetName 

**`MsgSetName`**

The naming convention for the SDK `Msgs` is `Msg{ .Action }`. The first action to implement is `SetName`, so we'll call it `MsgSetName`. This `Msg` allows the owner of a name to set the return value for that name within the resolver. Start by defining `MsgSetName` in the file called `./x/nameservice/types/msgs.go`:

```go
package types

import (
	sdk "github.com/cosmos/cosmos-sdk/types"
)

const RouterKey = ModuleName // this was defined in your key.go file

// MsgSetName defines a SetName message
type MsgSetName struct {
	Name  string         `json:"name"`
	Value string         `json:"value"`
	Owner sdk.AccAddress `json:"owner"`
}

// NewMsgSetName is a constructor function for MsgSetName
func NewMsgSetName(name string, value string, owner sdk.AccAddress) MsgSetName {
	return MsgSetName{
		Name:  name,
		Value: value,
		Owner: owner,
	}
}
```

The `MsgSetName` has the three attributes needed to set the value for a name:

* `name` - The name trying to be set.
* `value` - What the name resolves to.
* `owner` - The owner of that name.

Next, implement the `Msg` interface:

```go
// Route should return the name of the module
func (msg MsgSetName) Route() string { return RouterKey }

// Type should return the action
func (msg MsgSetName) Type() string { return "set_name" }
```

The above functions are used by the SDK to route `Msgs` to the proper module for handling. They also add human readable names to database tags used for indexing.

```go
// ValidateBasic runs stateless checks on the message
func (msg MsgSetName) ValidateBasic() error {
	if msg.Owner.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Owner.String())
	}
	if len(msg.Name) == 0 || len(msg.Value) == 0 {
		return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name and/or Value cannot be empty")
	}
	return nil
}
```

`ValidateBasic` is used to provide some basic **stateless** checks on the validity of the `Msg`. In this case, check that none of the attributes are empty.

```go
// GetSignBytes encodes the message for signing
func (msg MsgSetName) GetSignBytes() []byte {
	return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}
```

`GetSignBytes` defines how the `Msg` gets encoded for signing. In most cases this means marshal to sorted JSON. The output should not be modified.

```go
// GetSigners defines whose signature is required
func (msg MsgSetName) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.Owner}
}
```

`GetSigners` defines whose signature is required on a `Tx` in order for it to be valid. In this case, for example, the `MsgSetName` requires that the `Owner` signs the transaction when trying to reset what the name points to.

**`Handler`**

Now that `MsgSetName` is specified, the next step is to define what action\(s\) needs to be taken when this message is received. This is the role of the `handler`.

In the file (`./x/nameservice/handler.go`) start with the following code:

```go
package nameservice

import (
	"fmt"

	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"

	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
		switch msg := msg.(type) {
		case MsgSetName:
			return handleMsgSetName(ctx, keeper, msg)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
		}
	}
}
```

`NewHandler` is essentially a sub-router that directs messages coming into this module to the proper handler. At the moment, there is only one `Msg`/`Handler`.

Now, you need to define the actual logic for handling the `MsgSetName` message in `handleMsgSetName`:

> _NOTE_: The naming convention for handler names in the SDK is `handleMsg{ .Action }`

```go
// Handle a message to set name
func handleMsgSetName(ctx sdk.Context, keeper Keeper, msg MsgSetName) (*sdk.Result, error) {
	if !msg.Owner.Equals(keeper.GetOwner(ctx, msg.Name)) { // Checks if the the msg sender is the same as the current owner
		return nil, sdkerrors.Wrap(sdkerrors.ErrUnauthorized, "Incorrect Owner") // If not, throw an error
	}
	keeper.SetName(ctx, msg.Name, msg.Value) // If so, set the name to the value specified in the msg.
	return &sdk.Result{}, nil                // return
}
```

In this function, check to see if the `Msg` sender is actually the owner of the name \(`keeper.GetOwner`\). If so, they can set the name by calling the function on the `Keeper`. If not, throw an error and return that to the user.

Great, now owners can `SetName`s! But what if a name doesn't have an owner yet? Your module needs a way for users to buy names! Let us define the `BuyName` message. 

## MsgBuyName

Now it is time to define the `Msg` for buying names and add it to the `./x/nameservice/types/msgs.go` file. This code is very similar to `SetName`:

```go
// MsgBuyName defines the BuyName message
type MsgBuyName struct {
	Name  string         `json:"name"`
	Bid   sdk.Coins      `json:"bid"`
	Buyer sdk.AccAddress `json:"buyer"`
}

// NewMsgBuyName is the constructor function for MsgBuyName
func NewMsgBuyName(name string, bid sdk.Coins, buyer sdk.AccAddress) MsgBuyName {
	return MsgBuyName{
		Name:  name,
		Bid:   bid,
		Buyer: buyer,
	}
}

// Route should return the name of the module
func (msg MsgBuyName) Route() string { return RouterKey }

// Type should return the action
func (msg MsgBuyName) Type() string { return "buy_name" }

// ValidateBasic runs stateless checks on the message
func (msg MsgBuyName) ValidateBasic() error {
	if msg.Buyer.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Buyer.String())
	}
	if len(msg.Name) == 0 {
		return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
	}
	if !msg.Bid.IsAllPositive() {
		return sdkerrors.ErrInsufficientFunds
	}
	return nil
}

// GetSignBytes encodes the message for signing
func (msg MsgBuyName) GetSignBytes() []byte {
	return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners defines whose signature is required
func (msg MsgBuyName) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.Buyer}
}
```

Next, in the `./x/nameservice/handler.go` file, add the `MsgBuyName` handler to the module router:

```go
// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
		switch msg := msg.(type) {
		case MsgSetName:
			return handleMsgSetName(ctx, keeper, msg)
		case MsgBuyName:
			return handleMsgBuyName(ctx, keeper, msg)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
		}
	}
}
```

Finally, define the `BuyName` `handler` function which performs the state transitions triggered by the message. Keep in mind that at this point the message has had its `ValidateBasic` function run so there has been some input verification. However, `ValidateBasic` cannot query application state. Validation logic that is dependent on network state \(e.g. account balances\) should be performed in the `handler` function.

```go
// Handle a message to buy name
func handleMsgBuyName(ctx sdk.Context, keeper Keeper, msg MsgBuyName) (*sdk.Result, error) {
	// Checks if the the bid price is greater than the price paid by the current owner
	if keeper.GetPrice(ctx, msg.Name).IsAllGT(msg.Bid) {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInsufficientFunds, "Bid not high enough") // If not, throw an error
	}
	if keeper.HasOwner(ctx, msg.Name) {
		err := keeper.CoinKeeper.SendCoins(ctx, msg.Buyer, keeper.GetOwner(ctx, msg.Name), msg.Bid)
		if err != nil {
			return nil, err
		}
	} else {
		_, err := keeper.CoinKeeper.SubtractCoins(ctx, msg.Buyer, msg.Bid) // If so, deduct the Bid amount from the sender
		if err != nil {
			return nil, err
		}
	}
	keeper.SetOwner(ctx, msg.Name, msg.Buyer)
	keeper.SetPrice(ctx, msg.Name, msg.Bid)
	return &sdk.Result{}, nil
}
```

First check to make sure that the bid is higher than the current price. Then, check to see whether the name already has an owner. If it does, the former owner will receive the money from the `Buyer`.

If there is no owner, your `nameservice` module "burns" \(i.e. sends to an unrecoverable address\) the coins from the `Buyer`.

If either `SubtractCoins` or `SendCoins` returns a non-nil error, the handler throws an error, reverting the state transition. Otherwise, using the getters and setters defined on the `Keeper` earlier, the handler sets the buyer to the new owner and sets the new price to be the current bid.

> _NOTE_: This handler uses functions from the `coinKeeper` to perform currency operations. If your application is performing currency operations you may want to take a look at the [godocs for this module](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank#BaseKeeper) to see what functions it exposes.

Great, now owners can `BuyName`s! But what if they don't want the name any longer? Your module needs a way for users to delete names! Let us define the `DeleteName` message.

## MsgDeleteName 

Now it is time to define the `Msg` for deleting names and add it to the `./x/nameservice/types/msgs.go` file. This code is very similar to `SetName`:

```go
// MsgDeleteName defines a DeleteName message
type MsgDeleteName struct {
	Name  string         `json:"name"`
	Owner sdk.AccAddress `json:"owner"`
}

// NewMsgDeleteName is a constructor function for MsgDeleteName
func NewMsgDeleteName(name string, owner sdk.AccAddress) MsgDeleteName {
	return MsgDeleteName{
		Name:  name,
		Owner: owner,
	}
}

// Route should return the name of the module
func (msg MsgDeleteName) Route() string { return RouterKey }

// Type should return the action
func (msg MsgDeleteName) Type() string { return "delete_name" }

// ValidateBasic runs stateless checks on the message
func (msg MsgDeleteName) ValidateBasic() error {
	if msg.Owner.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Owner.String())
	}
	if len(msg.Name) == 0 {
		return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
	}
	return nil
}

// GetSignBytes encodes the message for signing
func (msg MsgDeleteName) GetSignBytes() []byte {
	return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners defines whose signature is required
func (msg MsgDeleteName) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.Owner}
}
```

Next, in the `./x/nameservice/handler.go` file, add the `MsgDeleteName` handler to the module router:

```go
// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
		switch msg := msg.(type) {
		case MsgSetName:
			return handleMsgSetName(ctx, keeper, msg)
		case MsgBuyName:
			return handleMsgBuyName(ctx, keeper, msg)
		case MsgDeleteName:
			return handleMsgDeleteName(ctx, keeper, msg)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
		}
	}
}
```

Finally, define the `DeleteName` `handler` function which performs the state transitions triggered by the message. Keep in mind that at this point the message has had its `ValidateBasic` function run so there has been some input verification. However, `ValidateBasic` cannot query application state. Validation logic that is dependent on network state \(e.g. account balances\) should be performed in the `handler` function.

```go
// Handle a message to delete name
func handleMsgDeleteName(ctx sdk.Context, keeper Keeper, msg MsgDeleteName) (*sdk.Result, error) {
	if !keeper.IsNamePresent(ctx, msg.Name) {
		return nil, sdkerrors.Wrap(types.ErrNameDoesNotExist, msg.Name)
	}
	if !msg.Owner.Equals(keeper.GetOwner(ctx, msg.Name)) {
		return nil, sdkerrors.Wrap(sdkerrors.ErrUnauthorized, "Incorrect Owner")
	}

	keeper.DeleteWhois(ctx, msg.Name)
	return &sdk.Result{}, nil
}
```

First check to see if the name currently exists in the store. If not, throw an error and return that to the user. Then check to see if the `Msg` sender is actually the owner of the name \(`keeper.GetOwner`\). If so, they can delete the name by calling the function on the `Keeper`. If not, throw an error and return that to the user.

Now that you have your `Msgs` and `Handlers` defined it's time to learn about making the data from these transactions available for querying. 

# Queriers 

**Query Types** 

Start by navigating to `./x/nameservice/types/querier.go` file. This is where you will define your querier types.

```go
package types

import "strings"

// QueryResResolve Queries Result Payload for a resolve query
type QueryResResolve struct {
	Value string `json:"value"`
}

// implement fmt.Stringer
func (r QueryResResolve) String() string {
	return r.Value
}

// QueryResNames Queries Result Payload for a names query
type QueryResNames []string

// implement fmt.Stringer
func (n QueryResNames) String() string {
	return strings.Join(n[:], "\n")
}
```

**Querier**

Now you can navigate to the `./x/nameservice/keeper/querier.go` file. This is the place to define which queries against application state users will be able to make. Your `nameservice` module will expose three queries:

* `resolve`: This takes a `name` and returns the `value` that is stored by the `nameservice`. This is similar to a DNS query.
* `whois`: This takes a `name` and returns the `price`, `value`, and `owner` of the name. Used for figuring out how much names cost when you want to buy them.
* `name` : This does not take a parameter, it returns all the names stored in the `nameservice` store.

You will see `NewQuerier` already defined, this function acts as a sub-router for queries to this module \(similar the `NewHandler` function\). Note that because there isn't an interface similar to `Msg` for queries, you need to manually define switch statement cases \(they can't be pulled off of the query `.Route()` function\):

```go
package keeper

import (
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"

	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/tendermint/abci/types"
)

// query endpoints supported by the nameservice Querier
const (
	QueryResolve = "resolve"
	QueryWhois   = "whois"
	QueryNames   = "names"
)

// NewQuerier is the module level router for state queries
func NewQuerier(keeper Keeper) sdk.Querier {
	return func(ctx sdk.Context, path []string, req abci.RequestQuery) (res []byte, err error) {
		switch path[0] {
		case QueryResolve:
			return queryResolve(ctx, path[1:], req, keeper)
		case QueryWhois:
			return queryWhois(ctx, path[1:], req, keeper)
		case QueryNames:
			return queryNames(ctx, req, keeper)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "unknown nameservice query endpoint")
		}
	}
}
```

Now that the router is defined, define the inputs and responses for each query: 

```go
// nolint: unparam
func queryResolve(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
	value := keeper.ResolveName(ctx, path[0])

	if value == "" {
		return []byte{}, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "could not resolve name")
	}

	res, err := codec.MarshalJSONIndent(keeper.cdc, types.QueryResResolve{Value: value})
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}

	return res, nil
}

// nolint: unparam
func queryWhois(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
	whois := keeper.GetWhois(ctx, path[0])

	res, err := codec.MarshalJSONIndent(keeper.cdc, whois)
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}

	return res, nil
}

func queryNames(ctx sdk.Context, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
	var namesList types.QueryResNames

	iterator := keeper.GetNamesIterator(ctx)

	for ; iterator.Valid(); iterator.Next() {
		namesList = append(namesList, string(iterator.Key()))
	}

	res, err := codec.MarshalJSONIndent(keeper.cdc, namesList)
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}

	return res, nil
}
```

Notes on the above code:

* Here your `Keeper`'s getters and setters come into heavy use. When building any other applications that use this module you may need to go back and define more getters/setters to access the pieces of state you need.
* By convention, each output type should be something that is both JSON marshalable and stringable \(implements the Golang `fmt.Stringer` interface\). The returned bytes should be the JSON encoding of the output result.
  * So for the output type of `resolve` we wrap the resolution string in a struct called `QueryResResolve` which is both JSON marshalable and has a `.String()` method.
  * For the output of Whois, the normal Whois struct is already JSON marshalable, but we need to add a `.String()` method on it.
  * Same for the output of a names query, a `[]string` is already natively marshalable, but we want to add a `.String()` method on it.
* The type Whois is not defined in the `./x/nameservice/types/querier.go` file because it is created in the `./x/nameservice/types/types.go` file.

Now that you have ways to mutate and view your module state it's time to put the finishing touches on it! Define the variables and types you would like to bring to the top level of the module. 

# Alias 

Start by navigating to the `./x/nameservice/alias.go` file. The main reason for having this file is to prevent import cycles. You can read more about import cycles in go here: [Golang import cycles](https://stackoverflow.com/questions/28256923/import-cycle-not-allowed)

First start by importing the "types" folder you have created.

 There are three kinds of types we will create in the alias.go file. 

* A constant, this is where you will define immutable variables.
* A variable, which you will define to contain information such as your messages.
* A type, here you will define the types you have created in the types folder.

```go
package nameservice

import (
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/keeper"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
)

const (
	ModuleName   = types.ModuleName
	RouterKey    = types.RouterKey
	StoreKey     = types.StoreKey
	QuerierRoute = types.QuerierRoute
)

var (
	NewKeeper        = keeper.NewKeeper
	NewQuerier       = keeper.NewQuerier
	NewMsgBuyName    = types.NewMsgBuyName
	NewMsgSetName    = types.NewMsgSetName
	NewMsgDeleteName = types.NewMsgDeleteName
	NewWhois         = types.NewWhois
	ModuleCdc        = types.ModuleCdc
	RegisterCodec    = types.RegisterCodec
)

type (
	Keeper          = keeper.Keeper
	MsgSetName      = types.MsgSetName
	MsgBuyName      = types.MsgBuyName
	MsgDeleteName   = types.MsgDeleteName
	QueryResResolve = types.QueryResResolve
	QueryResNames   = types.QueryResNames
	Whois           = types.Whois
)
```

Now you have aliased your needed constants, variables, and types. We can move forward with the creation of the module.

Register your types in the Amino encoding format next.

# Codec File

To [register your types with Amino](https://github.com/tendermint/go-amino#registering-types) so that they can be encoded/decoded, there is a bit of code that needs to be placed in `./x/nameservice/types/codec.go`. Any interface you create and any struct that implements an interface needs to be declared in the `RegisterCodec` function. In this module the three `Msg` implementations \(`SetName`, `BuyName` and `DeleteName`\) need to be registered, but your `Whois` query return type does not. In addition, we define a module specific codec for use later.

```go
package types

import (
	"github.com/cosmos/cosmos-sdk/codec"
)

// ModuleCdc is the codec for the module
var ModuleCdc = codec.New()

func init() {
	RegisterCodec(ModuleCdc)
}

// RegisterCodec registers concrete types on the Amino codec
func RegisterCodec(cdc *codec.Codec) {
	cdc.RegisterConcrete(MsgSetName{}, "nameservice/SetName", nil)
	cdc.RegisterConcrete(MsgBuyName{}, "nameservice/BuyName", nil)
	cdc.RegisterConcrete(MsgDeleteName{}, "nameservice/DeleteName", nil)
}
```

Next you need to define CLI interactions with your module.

## Nameservice Module CLI

The Cosmos SDK uses the [`cobra`](https://github.com/spf13/cobra) library for CLI interactions. This library makes it easy for each module to expose its own commands. To get started defining the user's CLI interactions with your module, create the following files:

* `./x/nameservice/client/cli/query.go`
* `./x/nameservice/client/cli/tx.go`

## Queries

Start in `query.go`. Here, define `cobra.Command`s for each of your modules `Queriers` \(`resolve`, and `whois`\):

```go
package cli

import (
	"fmt"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
	"github.com/spf13/cobra"
)

func GetQueryCmd(storeKey string, cdc *codec.Codec) *cobra.Command {
	nameserviceQueryCmd := &cobra.Command{
		Use:                        types.ModuleName,
		Short:                      "Querying commands for the nameservice module",
		DisableFlagParsing:         true,
		SuggestionsMinimumDistance: 2,
		RunE:                       client.ValidateCmd,
	}
	nameserviceQueryCmd.AddCommand(flags.GetCommands(
		GetCmdResolveName(storeKey, cdc),
		GetCmdWhois(storeKey, cdc),
		GetCmdNames(storeKey, cdc),
	)...)

	return nameserviceQueryCmd
}

// GetCmdResolveName queries information about a name
func GetCmdResolveName(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "resolve [name]",
		Short: "resolve name",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			name := args[0]

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/resolve/%s", queryRoute, name), nil)
			if err != nil {
				fmt.Printf("could not resolve name - %s \n", name)
				return nil
			}

			var out types.QueryResResolve
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}

// GetCmdWhois queries information about a domain
func GetCmdWhois(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "whois [name]",
		Short: "Query whois info of name",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			name := args[0]

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/whois/%s", queryRoute, name), nil)
			if err != nil {
				fmt.Printf("could not resolve whois - %s \n", name)
				return nil
			}

			var out types.Whois
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}

// GetCmdNames queries a list of all names
func GetCmdNames(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "names",
		Short: "names",
		// Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/names", queryRoute), nil)
			if err != nil {
				fmt.Printf("could not get query names\n")
				return nil
			}

			var out types.QueryResNames
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}
```

Notes on the above code:

* The CLI introduces a new `context`: [`CLIContext`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/context#CLIContext). It carries data about user input and application configuration that are needed for CLI interactions.
* The `path` required for the `cliCtx.QueryWithData()` function maps directly to the names in your query router.
  * The first part of the path is used to differentiate the types of queries possible to SDK applications: `custom` is for `Queriers`.
  * The second piece \(`nameservice`\) is the name of the module to route the query to.
  * Finally there is the specific querier in the module that will be called.
  * In this example the fourth piece is the query. This works because the query parameter is a simple string. To enable more complex query inputs you need to use the second argument of the [`.QueryWithData()`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/context#CLIContext.QueryWithData) function to pass in `data`. For an example of this see the [queriers in the Staking module](https://github.com/cosmos/cosmos-sdk/blob/5af6bd77aa6c0e8facc936947a3365416892e44d/x/staking/keeper/querier.go).

# Transactions

Now that the query interactions are defined, it is time to move on to transaction generation in `tx.go`:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`\).

```go
package cli

import (
	"bufio"

	"github.com/spf13/cobra"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/x/auth"
	"github.com/cosmos/cosmos-sdk/x/auth/client/utils"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"
)

func GetTxCmd(storeKey string, cdc *codec.Codec) *cobra.Command {
	nameserviceTxCmd := &cobra.Command{
		Use:                        types.ModuleName,
		Short:                      "Nameservice transaction subcommands",
		DisableFlagParsing:         true,
		SuggestionsMinimumDistance: 2,
		RunE:                       client.ValidateCmd,
	}

	nameserviceTxCmd.AddCommand(flags.PostCommands(
		GetCmdBuyName(cdc),
		GetCmdSetName(cdc),
		GetCmdDeleteName(cdc),
	)...)

	return nameserviceTxCmd
}

// GetCmdBuyName is the CLI command for sending a BuyName transaction
func GetCmdBuyName(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "buy-name [name] [amount]",
		Short: "bid for existing name or claim new name",
		Args:  cobra.ExactArgs(2),
		RunE: func(cmd *cobra.Command, args []string) error {
			inBuf := bufio.NewReader(cmd.InOrStdin())
			cliCtx := context.NewCLIContext().WithCodec(cdc)

			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			coins, err := sdk.ParseCoins(args[1])
			if err != nil {
				return err
			}

			msg := types.NewMsgBuyName(args[0], coins, cliCtx.GetFromAddress())
			err = msg.ValidateBasic()
			if err != nil {
				return err
			}

			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}

// GetCmdSetName is the CLI command for sending a SetName transaction
func GetCmdSetName(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "set-name [name] [value]",
		Short: "set the value associated with a name that you own",
		Args:  cobra.ExactArgs(2),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			inBuf := bufio.NewReader(cmd.InOrStdin())
			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			// if err := cliCtx.EnsureAccountExists(); err != nil {
			// 	return err
			// }

			msg := types.NewMsgSetName(args[0], args[1], cliCtx.GetFromAddress())
			err := msg.ValidateBasic()
			if err != nil {
				return err
			}

			// return utils.CompleteAndBroadcastTxCLI(txBldr, cliCtx, msgs)
			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}

// GetCmdDeleteName is the CLI command for sending a DeleteName transaction
func GetCmdDeleteName(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "delete-name [name]",
		Short: "delete the name that you own along with it's associated fields",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			inBuf := bufio.NewReader(cmd.InOrStdin())
			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			msg := types.NewMsgDeleteName(args[0], cliCtx.GetFromAddress())
			err := msg.ValidateBasic()
			if err != nil {
				return err
			}

			// return utils.CompleteAndBroadcastTxCLI(txBldr, cliCtx, msgs)
			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}
```

Notes on the above code:

* The `authcmd` package is used here. [The godocs have more information on usage](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth/client/cli#GetAccountDecoder). It provides access to accounts controlled by the CLI and facilitates signing.

Now you're ready to define the routes that the REST client will use to communicate with your module. 

## NameService Module Rest Interface 

Your module can also expose a REST interface to allow programatic access to the module's functionality. To get started navigate to `./x/nameservice/client/rest/rest.go` this file will hold the HTTP handlers:

Add in the `imports` and `const`s to get started:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`\).

```go
package rest

import (
	"fmt"

	"github.com/cosmos/cosmos-sdk/client/context"

	"github.com/gorilla/mux"
)

const (
	restName = "name"
)
```

## RegisterRoutes

First, define the REST client interface for your module in a `RegisterRoutes` function. Have the routes all start with your module name to prevent name space collisions with other modules' routes:

```go
// RegisterRoutes - Central function to define routes that get registered by the main application
func RegisterRoutes(cliCtx context.CLIContext, r *mux.Router, storeName string) {
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), namesHandler(cliCtx, storeName)).Methods("GET")
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), buyNameHandler(cliCtx)).Methods("POST")
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), setNameHandler(cliCtx)).Methods("PUT")
	r.HandleFunc(fmt.Sprintf("/%s/names/{%s}", storeName, restName), resolveNameHandler(cliCtx, storeName)).Methods("GET")
	r.HandleFunc(fmt.Sprintf("/%s/names/{%s}/whois", storeName, restName), whoIsHandler(cliCtx, storeName)).Methods("GET")
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), deleteNameHandler(cliCtx)).Methods("DELETE")
}
```

## Query Handlers

First create a `query.go` file to place all your querys in.

Next, its time to define the handlers mentioned above. These will be very similar to the CLI methods defined earlier. Start with the queries `whois` and `resolve`:

```go
package rest

import (
	"fmt"
	"net/http"

	"github.com/cosmos/cosmos-sdk/client/context"

	"github.com/cosmos/cosmos-sdk/types/rest"

	"github.com/gorilla/mux"
)

func resolveNameHandler(cliCtx context.CLIContext, storeName string) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		paramType := vars[restName]

		res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/resolve/%s", storeName, paramType), nil)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusNotFound, err.Error())
			return
		}

		rest.PostProcessResponse(w, cliCtx, res)
	}
}

func whoIsHandler(cliCtx context.CLIContext, storeName string) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		paramType := vars[restName]

		res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/whois/%s", storeName, paramType), nil)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusNotFound, err.Error())
			return
		}

		rest.PostProcessResponse(w, cliCtx, res)
	}
}

func namesHandler(cliCtx context.CLIContext, storeName string) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/names", storeName), nil)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusNotFound, err.Error())
			return
		}
		rest.PostProcessResponse(w, cliCtx, res)
	}
}
```

Notes on the above code:

* Notice we are using the same `cliCtx.QueryWithData` function to fetch the data
* These functions are almost the same as the corresponding CLI functionality

## Tx Handlers

First define a `tx.go` file to hold all your tx rest endpoints.

Now define the `buyName`, `setName` and `deleteName` transaction routes. Notice these aren't actually sending the transactions to buy, set and delete names. That would require sending a password along with the request which would be a security issue. Instead these endpoints build and return each specific transaction which can then be signed in a secure manner and afterwards broadcast to the network using a standard endpoint like `/txs`.

```go
package rest

import (
	"net/http"

	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/rest"
	"github.com/cosmos/cosmos-sdk/x/auth/client/utils"
)

type buyNameReq struct {
	BaseReq rest.BaseReq `json:"base_req"`
	Name    string       `json:"name"`
	Amount  string       `json:"amount"`
	Buyer   string       `json:"buyer"`
}

func buyNameHandler(cliCtx context.CLIContext) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		var req buyNameReq

		if !rest.ReadRESTReq(w, r, cliCtx.Codec, &req) {
			rest.WriteErrorResponse(w, http.StatusBadRequest, "failed to parse request")
			return
		}

		baseReq := req.BaseReq.Sanitize()
		if !baseReq.ValidateBasic(w) {
			return
		}

		addr, err := sdk.AccAddressFromBech32(req.Buyer)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		coins, err := sdk.ParseCoins(req.Amount)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		// create the message
		msg := types.NewMsgBuyName(req.Name, coins, addr)
		err = msg.ValidateBasic()
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		utils.WriteGenerateStdTxResponse(w, cliCtx, baseReq, []sdk.Msg{msg})
	}
}

type setNameReq struct {
	BaseReq rest.BaseReq `json:"base_req"`
	Name    string       `json:"name"`
	Value   string       `json:"value"`
	Owner   string       `json:"owner"`
}

func setNameHandler(cliCtx context.CLIContext) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		var req setNameReq
		if !rest.ReadRESTReq(w, r, cliCtx.Codec, &req) {
			rest.WriteErrorResponse(w, http.StatusBadRequest, "failed to parse request")
			return
		}

		baseReq := req.BaseReq.Sanitize()
		if !baseReq.ValidateBasic(w) {
			return
		}

		addr, err := sdk.AccAddressFromBech32(req.Owner)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		// create the message
		msg := types.NewMsgSetName(req.Name, req.Value, addr)
		err = msg.ValidateBasic()
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		utils.WriteGenerateStdTxResponse(w, cliCtx, baseReq, []sdk.Msg{msg})
	}
}

type deleteNameReq struct {
	BaseReq rest.BaseReq `json:"base_req"`
	Name    string       `json:"name"`
	Owner   string       `json:"owner"`
}

func deleteNameHandler(cliCtx context.CLIContext) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		var req deleteNameReq
		if !rest.ReadRESTReq(w, r, cliCtx.Codec, &req) {
			rest.WriteErrorResponse(w, http.StatusBadRequest, "failed to parse request")
			return
		}

		baseReq := req.BaseReq.Sanitize()
		if !baseReq.ValidateBasic(w) {
			return
		}

		addr, err := sdk.AccAddressFromBech32(req.Owner)
		if err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		// create the message
		msg := types.NewMsgDeleteName(req.Name, addr)
		err = msg.ValidateBasic()
		if err := msg.ValidateBasic(); err != nil {
			rest.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		utils.WriteGenerateStdTxResponse(w, cliCtx, baseReq, []sdk.Msg{msg})
	}
}
```

Notes on the above code:

* The [`BaseReq`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/utils#BaseReq) contains the basic required fields for making a transaction \(which key to use, how to decode it, which chain you are on, etc...\) and is designed to be embedded as shown.
* `baseReq.ValidateBasic` handles setting the response code for you and therefore you don't need to worry about handling errors or successes when using those functions.

Next its time to augment `nameservice` by implementing the AppModule interface.

# AppModule Interface

The Cosmos SDK provides a standard interface for modules. This [`AppModule`](https://github.com/cosmos/cosmos-sdk/blob/master/types/module.go) interface requires modules to provide a set of methods used by the `ModuleBasicsManager` to incorporate them into your application. First we will scaffold out the interface and implement **some** of its methods. Then we will incorporate our nameservice module alongside `auth` and `bank` into our app.

Start by opening two new files, `module.go` and `genesis.go`. We will implement the AppModule interface in `module.go` and the functions specific to genesis state management in `genesis.go`. The genesis-specific methods on your AppModule struct will be pass-though calls to those defined in `genesis.go`.

Lets start with adding the following code to `module.go`. The scaffolding tool has already filled in all the functions with the data that is needed but there are still a couple of to-dos. You will have to add the modules that this module depends on in the `AppModule` type.

```go
package nameservice

import (
	"encoding/json"

	"github.com/gorilla/mux"
	"github.com/spf13/cobra"

	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/cosmos-sdk/types/module"
	"github.com/cosmos/cosmos-sdk/x/bank"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/client/cli"
	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/client/rest"

	"github.com/cosmos/cosmos-sdk/client/context"
	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/tendermint/abci/types"
)

// type check to ensure the interface is properly implemented
var (
	_ module.AppModule      = AppModule{}
	_ module.AppModuleBasic = AppModuleBasic{}
)

// app module Basics object
type AppModuleBasic struct{}

func (AppModuleBasic) Name() string {
	return ModuleName
}

func (AppModuleBasic) RegisterCodec(cdc *codec.Codec) {
	RegisterCodec(cdc)
}

func (AppModuleBasic) DefaultGenesis() json.RawMessage {
	return ModuleCdc.MustMarshalJSON(DefaultGenesisState())
}

// Validation check of the Genesis
func (AppModuleBasic) ValidateGenesis(bz json.RawMessage) error {
	var data GenesisState
	err := ModuleCdc.UnmarshalJSON(bz, &data)
	if err != nil {
		return err
	}
	// Once json successfully marshalled, passes along to genesis.go
	return ValidateGenesis(data)
}

// Register rest routes
func (AppModuleBasic) RegisterRESTRoutes(ctx context.CLIContext, rtr *mux.Router) {
	rest.RegisterRoutes(ctx, rtr, StoreKey)
}

// Get the root query command of this module
func (AppModuleBasic) GetQueryCmd(cdc *codec.Codec) *cobra.Command {
	return cli.GetQueryCmd(StoreKey, cdc)
}

// Get the root tx command of this module
func (AppModuleBasic) GetTxCmd(cdc *codec.Codec) *cobra.Command {
	return cli.GetTxCmd(StoreKey, cdc)
}

type AppModule struct {
	AppModuleBasic
	keeper     Keeper
	bankKeeper bank.Keeper
}

// NewAppModule creates a new AppModule Object
func NewAppModule(k Keeper, bankKeeper bank.Keeper) AppModule {
	return AppModule{
		AppModuleBasic: AppModuleBasic{},
		keeper:         k,
		bankKeeper:     bankKeeper,
	}
}

func (AppModule) Name() string {
	return ModuleName
}

func (am AppModule) RegisterInvariants(ir sdk.InvariantRegistry) {}

func (am AppModule) Route() string {
	return RouterKey
}

func (am AppModule) NewHandler() sdk.Handler {
	return NewHandler(am.keeper)
}
func (am AppModule) QuerierRoute() string {
	return QuerierRoute
}

func (am AppModule) NewQuerierHandler() sdk.Querier {
	return NewQuerier(am.keeper)
}

func (am AppModule) BeginBlock(_ sdk.Context, _ abci.RequestBeginBlock) {}

func (am AppModule) EndBlock(sdk.Context, abci.RequestEndBlock) []abci.ValidatorUpdate {
	return []abci.ValidatorUpdate{}
}

func (am AppModule) InitGenesis(ctx sdk.Context, data json.RawMessage) []abci.ValidatorUpdate {
	var genesisState GenesisState
	ModuleCdc.MustUnmarshalJSON(data, &genesisState)
	InitGenesis(ctx, am.keeper, genesisState)
	return []abci.ValidatorUpdate{}
}

func (am AppModule) ExportGenesis(ctx sdk.Context) json.RawMessage {
	gs := ExportGenesis(ctx, am.keeper)
	return ModuleCdc.MustMarshalJSON(gs)
}
```

To see more examples of AppModule implementation, check out some of the other modules in the SDK such as [x/staking](https://github.com/cosmos/cosmos-sdk/blob/master/x/staking/genesis.go)

Next, we need to implement the genesis-specific methods called above. 

## Genesis

The AppModule interface includes a number of functions for use in initializing and exporting GenesisState for the chain. The `ModuleBasicManager` calls these functions on each module when starting, stopping or exporting the chain. Here is a very basic implementation that you can expand upon.

Go to `x/nameservice/genesis.go` and you will see that a few things are missing. We have to fill them in according to the needs of the module. Below you will see what is missing:

```go
package nameservice

import (
	"fmt"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

type GenesisState struct {
	WhoisRecords []Whois `json:"whois_records"`
}

func NewGenesisState(whoIsRecords []Whois) GenesisState {
	return GenesisState{WhoisRecords: nil}
}

func ValidateGenesis(data GenesisState) error {
	for _, record := range data.WhoisRecords {
		if record.Owner == nil {
			return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Owner", record.Value)
		}
		if record.Value == "" {
			return fmt.Errorf("invalid WhoisRecord: Owner: %s. Error: Missing Value", record.Owner)
		}
		if record.Price == nil {
			return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Price", record.Value)
		}
	}
	return nil
}

func DefaultGenesisState() GenesisState {
	return GenesisState{
		WhoisRecords: []Whois{},
	}
}

func InitGenesis(ctx sdk.Context, keeper Keeper, data GenesisState) {
	for _, record := range data.WhoisRecords {
		keeper.SetWhois(ctx, record.Value, record)
	}
}

func ExportGenesis(ctx sdk.Context, k Keeper) GenesisState {
	var records []Whois
	iterator := k.GetNamesIterator(ctx)
	for ; iterator.Valid(); iterator.Next() {

		name := string(iterator.Key())
		whois := k.GetWhois(ctx, name)
		records = append(records, whois)

	}
	return GenesisState{WhoisRecords: records}
}
```

Next we will define what the genesis state will be, the default genesis and a way to validate it so we don't run into any errors when we start the chain with preexisting state.

```go
package types

import (
	"fmt"
)

type GenesisState struct {
	WhoisRecords []Whois `json:"whois_records"`
}

func NewGenesisState(whoIsRecords []Whois) GenesisState {
	return GenesisState{WhoisRecords: nil}
}

func ValidateGenesis(data GenesisState) error {
	for _, record := range data.WhoisRecords {
		if record.Owner == nil {
			return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Owner", record.Value)
		}
		if record.Value == "" {
			return fmt.Errorf("invalid WhoisRecord: Owner: %s. Error: Missing Value", record.Owner)
		}
		if record.Price == nil {
			return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Price", record.Value)
		}
	}
	return nil
}

func DefaultGenesisState() GenesisState {
	return GenesisState{
		WhoisRecords: []Whois{},
	}
}
```

A few notes about the above code:

* `ValidateGenesis()` validates the provided genesis state to ensure that expected invariants hold
* `DefaultGenesisState()` is used mostly for testing. This provides a minimal GenesisState.
* `InitGenesis()` is called on chain start, this function imports genesis state into the keeper.
* `ExportGenesis()` is called after stopping the chain, this function loads application state into a GenesisState struct to later be exported to `genesis.json` alongside data from the other modules.

Now your module has everything it needs to be incorporated into your Cosmos SDK application.

# Complete App

Now that your module is ready, it can be incorporated in the `./app.go` file. Let's begin by adding your new nameservice module to the imports:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`\).

```go
package app

import (
	"encoding/json"
	"os"

	abci "github.com/tendermint/tendermint/abci/types"
	"github.com/tendermint/tendermint/libs/log"
	tmos "github.com/tendermint/tendermint/libs/os"
	tmtypes "github.com/tendermint/tendermint/types"
	dbm "github.com/tendermint/tm-db"

	bam "github.com/cosmos/cosmos-sdk/baseapp"
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/cosmos-sdk/simapp"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/module"
	"github.com/cosmos/cosmos-sdk/version"
	"github.com/cosmos/cosmos-sdk/x/auth"
	"github.com/cosmos/cosmos-sdk/x/auth/vesting"
	"github.com/cosmos/cosmos-sdk/x/bank"
	distr "github.com/cosmos/cosmos-sdk/x/distribution"
	"github.com/cosmos/cosmos-sdk/x/genutil"
	"github.com/cosmos/cosmos-sdk/x/params"
	"github.com/cosmos/cosmos-sdk/x/slashing"
	"github.com/cosmos/cosmos-sdk/x/staking"
	"github.com/cosmos/cosmos-sdk/x/supply"

	"github.com/cosmos/sdk-tutorials/nameservice/x/nameservice"
)

const appName = "nameservice"

var (
	// default home directories for the application CLI
	DefaultCLIHome = os.ExpandEnv("$HOME/.nscli")

	// DefaultNodeHome sets the folder where the applcation data and configuration will be stored
	DefaultNodeHome = os.ExpandEnv("$HOME/.nsd")

	// NewBasicManager is in charge of setting up basic module elements
	ModuleBasics = module.NewBasicManager(
		genutil.AppModuleBasic{},
		auth.AppModuleBasic{},
		bank.AppModuleBasic{},
		staking.AppModuleBasic{},
		distr.AppModuleBasic{},
		params.AppModuleBasic{},
		slashing.AppModuleBasic{},
		supply.AppModuleBasic{},

		nameservice.AppModule{},
	)
	// account permissions
	maccPerms = map[string][]string{
		auth.FeeCollectorName:     nil,
		distr.ModuleName:          nil,
		staking.BondedPoolName:    {supply.Burner, supply.Staking},
		staking.NotBondedPoolName: {supply.Burner, supply.Staking},
	}
)
```

Next you need to add the stores' keys and the `Keepers` into your `nameServiceApp` struct.

```go
type nameServiceApp struct {
	*bam.BaseApp
	cdc *codec.Codec

	// keys to access the substores
	keys  map[string]*sdk.KVStoreKey
	tkeys map[string]*sdk.TransientStoreKey

	// subspaces
	subspaces map[string]params.Subspace

	// Keepers
	accountKeeper  auth.AccountKeeper
	bankKeeper     bank.Keeper
	stakingKeeper  staking.Keeper
	slashingKeeper slashing.Keeper
	distrKeeper    distr.Keeper
	supplyKeeper   supply.Keeper
	paramsKeeper   params.Keeper
	nsKeeper       nameservice.Keeper

	// Module Manager
	mm *module.Manager

	// simulation manager
	sm *module.SimulationManager
}

// verify app interface at compile time
var _ simapp.App = (*nameServiceApp)(nil)

// NewNameServiceApp is a constructor function for nameServiceApp
func NewNameServiceApp(
	logger log.Logger, db dbm.DB, baseAppOptions ...func(*bam.BaseApp),
) *nameServiceApp {

	// First define the top level codec that will be shared by the different modules
	cdc := MakeCodec()

	// BaseApp handles interactions with Tendermint through the ABCI protocol
	bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

	bApp.SetAppVersion(version.Version)

	keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
		supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, nameservice.StoreKey)

	tkeys := sdk.NewTransientStoreKeys(params.TStoreKey)

	// Here you initialize your application with the store keys it requires
	var app = &nameServiceApp{
		BaseApp:   bApp,
		cdc:       cdc,
		keys:      keys,
		tkeys:     tkeys,
		subspaces: make(map[string]params.Subspace),
	}
}
```

At this point, the constructor still lacks important logic. Namely, it needs to:

* Instantiate required `Keepers` from each desired module.
* Generate `storeKeys` required by each `Keeper`.
* Register `Handler`s from each module. The `AddRoute()` method from `baseapp`'s `router` is used to this end.
* Register `Querier`s from each module. The `AddRoute()` method from `baseapp`'s `queryRouter` is used to this end.
* Mount `KVStore`s to the provided keys in the `baseApp` multistore.
* Set the `initChainer` for defining the initial application state.

Your finalized constructor should look like this:

```go
// NewNameServiceApp is a constructor function for nameServiceApp
func NewNameServiceApp(
	logger log.Logger, db dbm.DB, baseAppOptions ...func(*bam.BaseApp),
) *nameServiceApp {

	// First define the top level codec that will be shared by the different modules
	cdc := MakeCodec()

	// BaseApp handles interactions with Tendermint through the ABCI protocol
	bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

	bApp.SetAppVersion(version.Version)

	keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
		supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, nameservice.StoreKey)

	tkeys := sdk.NewTransientStoreKeys(params.TStoreKey)

	// Here you initialize your application with the store keys it requires
	var app = &nameServiceApp{
		BaseApp:   bApp,
		cdc:       cdc,
		keys:      keys,
		tkeys:     tkeys,
		subspaces: make(map[string]params.Subspace),
	}

	// The ParamsKeeper handles parameter storage for the application
	app.paramsKeeper = params.NewKeeper(app.cdc, keys[params.StoreKey], tkeys[params.TStoreKey])
	// Set specific supspaces
	app.subspaces[auth.ModuleName] = app.paramsKeeper.Subspace(auth.DefaultParamspace)
	app.subspaces[bank.ModuleName] = app.paramsKeeper.Subspace(bank.DefaultParamspace)
	app.subspaces[staking.ModuleName] = app.paramsKeeper.Subspace(staking.DefaultParamspace)
	app.subspaces[distr.ModuleName] = app.paramsKeeper.Subspace(distr.DefaultParamspace)
	app.subspaces[slashing.ModuleName] = app.paramsKeeper.Subspace(slashing.DefaultParamspace)

	// The AccountKeeper handles address -> account lookups
	app.accountKeeper = auth.NewAccountKeeper(
		app.cdc,
		keys[auth.StoreKey],
		app.subspaces[auth.ModuleName],
		auth.ProtoBaseAccount,
	)

	// The BankKeeper allows you perform sdk.Coins interactions
	app.bankKeeper = bank.NewBaseKeeper(
		app.accountKeeper,
		app.subspaces[bank.ModuleName],
		app.ModuleAccountAddrs(),
	)

	// The SupplyKeeper collects transaction fees and renders them to the fee distribution module
	app.supplyKeeper = supply.NewKeeper(
		app.cdc,
		keys[supply.StoreKey],
		app.accountKeeper,
		app.bankKeeper,
		maccPerms,
	)

	// The staking keeper
	stakingKeeper := staking.NewKeeper(
		app.cdc,
		keys[staking.StoreKey],
		app.supplyKeeper,
		app.subspaces[staking.ModuleName],
	)

	app.distrKeeper = distr.NewKeeper(
		app.cdc,
		keys[distr.StoreKey],
		app.subspaces[distr.ModuleName],
		&stakingKeeper,
		app.supplyKeeper,
		auth.FeeCollectorName,
		app.ModuleAccountAddrs(),
	)

	app.slashingKeeper = slashing.NewKeeper(
		app.cdc,
		keys[slashing.StoreKey],
		&stakingKeeper,
		app.subspaces[slashing.ModuleName],
	)

	// register the staking hooks
	// NOTE: stakingKeeper above is passed by reference, so that it will contain these hooks
	app.stakingKeeper = *stakingKeeper.SetHooks(
		staking.NewMultiStakingHooks(
			app.distrKeeper.Hooks(),
			app.slashingKeeper.Hooks()),
	)

	// The NameserviceKeeper is the Keeper from the module for this tutorial
	// It handles interactions with the namestore
	app.nsKeeper = nameservice.NewKeeper(
		app.cdc,
		keys[nameservice.StoreKey],
		app.bankKeeper,
	)

	app.mm = module.NewManager(
		genutil.NewAppModule(app.accountKeeper, app.stakingKeeper, app.BaseApp.DeliverTx),
		auth.NewAppModule(app.accountKeeper),
		bank.NewAppModule(app.bankKeeper, app.accountKeeper),
		nameservice.NewAppModule(app.nsKeeper, app.bankKeeper),
		supply.NewAppModule(app.supplyKeeper, app.accountKeeper),
		distr.NewAppModule(app.distrKeeper, app.accountKeeper, app.supplyKeeper, app.stakingKeeper),
		slashing.NewAppModule(app.slashingKeeper, app.accountKeeper, app.stakingKeeper),
		staking.NewAppModule(app.stakingKeeper, app.accountKeeper, app.supplyKeeper),
	)

	app.mm.SetOrderBeginBlockers(distr.ModuleName, slashing.ModuleName)
	app.mm.SetOrderEndBlockers(staking.ModuleName)

	// Sets the order of Genesis - Order matters, genutil is to always come last
	// NOTE: The genutils moodule must occur after staking so that pools are
	// properly initialized with tokens from genesis accounts.
	app.mm.SetOrderInitGenesis(
		distr.ModuleName,
		staking.ModuleName,
		auth.ModuleName,
		bank.ModuleName,
		slashing.ModuleName,
		nameservice.ModuleName,
		supply.ModuleName,
		genutil.ModuleName,
	)

	// register all module routes and module queriers
	app.mm.RegisterRoutes(app.Router(), app.QueryRouter())

	// The initChainer handles translating the genesis.json file into initial state for the network
	app.SetInitChainer(app.InitChainer)
	app.SetBeginBlocker(app.BeginBlocker)
	app.SetEndBlocker(app.EndBlocker)

	// The AnteHandler handles signature verification and transaction pre-processing
	app.SetAnteHandler(
		auth.NewAnteHandler(
			app.accountKeeper,
			app.supplyKeeper,
			auth.DefaultSigVerificationGasConsumer,
		),
	)

	// initialize stores
	app.MountKVStores(keys)
	app.MountTransientStores(tkeys)

	err := app.LoadLatestVersion(app.keys[bam.MainStoreKey])
	if err != nil {
		tmos.Exit(err.Error())
	}

	return app
}
```

> _NOTE_: The TransientStore mentioned above is an in-memory implementation of the KVStore for state that is not persisted.
>
> Pay attention to how the modules are initiated: the order matters! Here the sequence goes Auth --&gt; Bank --&gt; Feecollection --&gt; Staking --&gt; Distribution --&gt; Slashing, then the hooks were set for the staking module. This is because some of these modules depend on others existing before they can be used.

The `initChainer` defines how accounts in `genesis.json` are mapped into the application state on initial chain start. The `ExportAppStateAndValidators` function helps bootstrap the initial state for the application. You don't need to worry too much about either of these for now. We also need to add a few more methods to our app `BeginBlocker`, `EndBlocker` and `LoadHeight`.

The constructor registers the `initChainer` function, but it isn't defined yet. Go ahead and create it:

```go
// GenesisState represents chain state at the start of the chain. Any initial state (account balances) are stored here.
type GenesisState map[string]json.RawMessage

func NewDefaultGenesisState() GenesisState {
	return ModuleBasics.DefaultGenesis()
}

func (app *nameServiceApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
	var genesisState GenesisState

	err := app.cdc.UnmarshalJSON(req.AppStateBytes, &genesisState)
	if err != nil {
		panic(err)
	}

	return app.mm.InitGenesis(ctx, genesisState)
}

func (app *nameServiceApp) BeginBlocker(ctx sdk.Context, req abci.RequestBeginBlock) abci.ResponseBeginBlock {
	return app.mm.BeginBlock(ctx, req)
}

func (app *nameServiceApp) EndBlocker(ctx sdk.Context, req abci.RequestEndBlock) abci.ResponseEndBlock {
	return app.mm.EndBlock(ctx, req)
}

// GetKey returns the KVStoreKey for the provided store key
func (app *nameServiceApp) GetKey(storeKey string) *sdk.KVStoreKey {
	return app.keys[storeKey]
}

// GetTKey returns the TransientStoreKey for the provided store key
func (app *nameServiceApp) GetTKey(storeKey string) *sdk.TransientStoreKey {
	return app.tkeys[storeKey]
}

func (app *nameServiceApp) LoadHeight(height int64) error {
	return app.LoadVersion(height, app.keys[bam.MainStoreKey])
}

// Codec returns simapp's codec
func (app *nameServiceApp) Codec() *codec.Codec {
	return app.cdc
}

// SimulationManager implements the SimulationApp interface
func (app *nameServiceApp) SimulationManager() *module.SimulationManager {
	return app.sm
}

// ModuleAccountAddrs returns all the app's module account addresses.
func (app *nameServiceApp) ModuleAccountAddrs() map[string]bool {
	modAccAddrs := make(map[string]bool)
	for acc := range maccPerms {
		modAccAddrs[supply.NewModuleAddress(acc).String()] = true
	}

	return modAccAddrs
}

//_________________________________________________________

func (app *nameServiceApp) ExportAppStateAndValidators(forZeroHeight bool, jailWhiteList []string,
) (appState json.RawMessage, validators []tmtypes.GenesisValidator, err error) {

	// as if they could withdraw from the start of the next block
	ctx := app.NewContext(true, abci.Header{Height: app.LastBlockHeight()})

	genState := app.mm.ExportGenesis(ctx)
	appState, err = codec.MarshalJSONIndent(app.cdc, genState)
	if err != nil {
		return nil, nil, err
	}

	validators = staking.WriteValidators(ctx, app.stakingKeeper)

	return appState, validators, nil
}
```

Finally add a helper function to generate an amino [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec) that properly registers all of the modules used in your application:

```go
// MakeCodec generates the necessary codecs for Amino
func MakeCodec() *codec.Codec {
	var cdc = codec.New()

	ModuleBasics.RegisterCodec(cdc)
	vesting.RegisterCodec(cdc)
	sdk.RegisterCodec(cdc)
	codec.RegisterCrypto(cdc)

	return cdc
}
```

Now that you have created an application that includes your module, it's time to build your entry points.

# Entrypoints 

In Golang the convention is to place files that compile to a binary in the `./cmd` folder of a project. For your application there are 2 binaries that you want to create:

* `nsd`: This binary is similar to `bitcoind` or other cryptocurrency daemons in that it maintains p2p connections, propagates transactions, handles local storage and provides an RPC interface to interact with the network. In this case, Tendermint is used for networking and transaction ordering.
* `nscli`: This binary provides commands that allow users to interact with your application.

To get started create two files in your project directory that will instantiate these binaries:

* `./cmd/nsd/main.go`
* `./cmd/nscli/main.go`

**`nsd`**

Start by adding the following code to `cmd/nsd/main.go`:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

```go
package main

import (
	"encoding/json"
	"io"

	"github.com/cosmos/cosmos-sdk/server"
	"github.com/cosmos/cosmos-sdk/x/staking"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
	"github.com/tendermint/tendermint/libs/cli"
	"github.com/tendermint/tendermint/libs/log"

	"github.com/cosmos/cosmos-sdk/baseapp"
	"github.com/cosmos/cosmos-sdk/client/debug"
	"github.com/cosmos/cosmos-sdk/client/flags"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/x/auth"
	genutilcli "github.com/cosmos/cosmos-sdk/x/genutil/client/cli"
	app "github.com/cosmos/sdk-tutorials/nameservice"
	abci "github.com/tendermint/tendermint/abci/types"
	tmtypes "github.com/tendermint/tendermint/types"
	dbm "github.com/tendermint/tm-db"
)

func main() {
	cobra.EnableCommandSorting = false

	cdc := app.MakeCodec()

	config := sdk.GetConfig()
	config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
	config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
	config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
	config.Seal()

	ctx := server.NewDefaultContext()

	rootCmd := &cobra.Command{
		Use:               "nsd",
		Short:             "nameservice App Daemon (server)",
		PersistentPreRunE: server.PersistentPreRunEFn(ctx),
	}
	// CLI commands to initialize the chain
	rootCmd.AddCommand(genutilcli.InitCmd(ctx, cdc, app.ModuleBasics, app.DefaultNodeHome))
	rootCmd.AddCommand(genutilcli.CollectGenTxsCmd(ctx, cdc, auth.GenesisAccountIterator{}, app.DefaultNodeHome))
	rootCmd.AddCommand(genutilcli.MigrateGenesisCmd(ctx, cdc))
	rootCmd.AddCommand(
		genutilcli.GenTxCmd(
			ctx, cdc, app.ModuleBasics, staking.AppModuleBasic{},
			auth.GenesisAccountIterator{}, app.DefaultNodeHome, app.DefaultCLIHome,
		),
	)
	rootCmd.AddCommand(genutilcli.ValidateGenesisCmd(ctx, cdc, app.ModuleBasics))
	// AddGenesisAccountCmd allows users to add accounts to the genesis file
	rootCmd.AddCommand(AddGenesisAccountCmd(ctx, cdc, app.DefaultNodeHome, app.DefaultCLIHome))
	rootCmd.AddCommand(flags.NewCompletionCmd(rootCmd, true))
	rootCmd.AddCommand(debug.Cmd(cdc))

	server.AddCommands(ctx, cdc, rootCmd, newApp, exportAppStateAndTMValidators)

	// prepare and add flags
	executor := cli.PrepareBaseCmd(rootCmd, "NS", app.DefaultNodeHome)
	err := executor.Execute()
	if err != nil {
		panic(err)
	}
}

func newApp(logger log.Logger, db dbm.DB, traceStore io.Writer) abci.Application {
	return app.NewNameServiceApp(logger, db, baseapp.SetMinGasPrices(viper.GetString(server.FlagMinGasPrices)))
}

func exportAppStateAndTMValidators(
	logger log.Logger, db dbm.DB, traceStore io.Writer, height int64, forZeroHeight bool, jailWhiteList []string,
) (json.RawMessage, []tmtypes.GenesisValidator, error) {

	if height != -1 {
		nsApp := app.NewNameServiceApp(logger, db)
		err := nsApp.LoadHeight(height)
		if err != nil {
			return nil, nil, err
		}
		return nsApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
	}

	nsApp := app.NewNameServiceApp(logger, db)

	return nsApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
}
```

Notes on the above code:

* Most of the code above combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.

**`nscli`**

Finish up by building the `nscli` command:

> _NOTE_: Your application needs to import the code you just wrote. Here the import path is set to this repository \(`github.com/cosmos/sdk-tutorials/nameservice`\). If you are following along in your own repo you will need to change the import path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

```go
package main

import (
	"os"
	"path"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/client/keys"
	"github.com/cosmos/cosmos-sdk/client/lcd"
	"github.com/cosmos/cosmos-sdk/client/rpc"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/version"
	"github.com/cosmos/cosmos-sdk/x/auth"
	authcmd "github.com/cosmos/cosmos-sdk/x/auth/client/cli"
	authrest "github.com/cosmos/cosmos-sdk/x/auth/client/rest"
	"github.com/cosmos/cosmos-sdk/x/bank"
	bankcmd "github.com/cosmos/cosmos-sdk/x/bank/client/cli"
	app "github.com/cosmos/sdk-tutorials/nameservice"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
	amino "github.com/tendermint/go-amino"
	"github.com/tendermint/tendermint/libs/cli"
)

func main() {
	cobra.EnableCommandSorting = false

	cdc := app.MakeCodec()

	// Read in the configuration file for the sdk
	config := sdk.GetConfig()
	config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
	config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
	config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
	config.Seal()

	rootCmd := &cobra.Command{
		Use:   "nscli",
		Short: "nameservice Client",
	}

	// Add --chain-id to persistent flags and mark it required
	rootCmd.PersistentFlags().String(flags.FlagChainID, "", "Chain ID of tendermint node")
	rootCmd.PersistentPreRunE = func(_ *cobra.Command, _ []string) error {
		return initConfig(rootCmd)
	}

	// Construct Root Command
	rootCmd.AddCommand(
		rpc.StatusCommand(),
		client.ConfigCmd(app.DefaultCLIHome),
		queryCmd(cdc),
		txCmd(cdc),
		flags.LineBreak,
		lcd.ServeCommand(cdc, registerRoutes),
		flags.LineBreak,
		keys.Commands(),
		flags.LineBreak,
		version.Cmd,
		flags.NewCompletionCmd(rootCmd, true),
	)

	executor := cli.PrepareMainCmd(rootCmd, "NS", app.DefaultCLIHome)
	err := executor.Execute()

	if err != nil {
		panic(err)
	}
}

func registerRoutes(rs *lcd.RestServer) {
	client.RegisterRoutes(rs.CliCtx, rs.Mux)
	authrest.RegisterTxRoutes(rs.CliCtx, rs.Mux)
	app.ModuleBasics.RegisterRESTRoutes(rs.CliCtx, rs.Mux)
}

func queryCmd(cdc *amino.Codec) *cobra.Command {
	queryCmd := &cobra.Command{
		Use:     "query",
		Aliases: []string{"q"},
		Short:   "Querying subcommands",
	}

	queryCmd.AddCommand(
		authcmd.GetAccountCmd(cdc),
		flags.LineBreak,
		rpc.ValidatorCommand(cdc),
		rpc.BlockCommand(),
		authcmd.QueryTxsByEventsCmd(cdc),
		authcmd.QueryTxCmd(cdc),
		flags.LineBreak,
	)

	// add modules' query commands
	app.ModuleBasics.AddQueryCommands(queryCmd, cdc)

	return queryCmd
}

func txCmd(cdc *amino.Codec) *cobra.Command {
	txCmd := &cobra.Command{
		Use:   "tx",
		Short: "Transactions subcommands",
	}

	txCmd.AddCommand(
		bankcmd.SendTxCmd(cdc),
		flags.LineBreak,
		authcmd.GetSignCommand(cdc),
		authcmd.GetMultiSignCommand(cdc),
		flags.LineBreak,
		authcmd.GetBroadcastCommand(cdc),
		authcmd.GetEncodeCommand(cdc),
		authcmd.GetDecodeCommand(cdc),
		flags.LineBreak,
	)

	// add modules' tx commands
	app.ModuleBasics.AddTxCommands(txCmd, cdc)

	// remove auth and bank commands as they're mounted under the root tx command
	var cmdsToRemove []*cobra.Command

	for _, cmd := range txCmd.Commands() {
		if cmd.Use == auth.ModuleName || cmd.Use == bank.ModuleName {
			cmdsToRemove = append(cmdsToRemove, cmd)
		}
	}

	txCmd.RemoveCommand(cmdsToRemove...)

	return txCmd
}

func initConfig(cmd *cobra.Command) error {
	home, err := cmd.PersistentFlags().GetString(cli.HomeFlag)
	if err != nil {
		return err
	}

	cfgFile := path.Join(home, "config", "config.toml")
	if _, err := os.Stat(cfgFile); err == nil {
		viper.SetConfigFile(cfgFile)

		if err := viper.ReadInConfig(); err != nil {
			return err
		}
	}
	if err := viper.BindPFlag(flags.FlagChainID, cmd.PersistentFlags().Lookup(flags.FlagChainID)); err != nil {
		return err
	}
	if err := viper.BindPFlag(cli.EncodingFlag, cmd.PersistentFlags().Lookup(cli.EncodingFlag)); err != nil {
		return err
	}
	return viper.BindPFlag(cli.OutputFlag, cmd.PersistentFlags().Lookup(cli.OutputFlag))
}
```

Note:

* The code combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.
* The [`cobra` CLI documentation](http://github.com/spf13/cobra) will help with understanding the above code.
* You can see the `ModuleClient` defined earlier in action here.
* Note how the routes are included in the `registerRoutes` function.

Now that you have your binaries defined its time to deal with dependency management and build your app.

## go.mod and Makefile 

**`Makefile`**

Help users build your application by writing a `./Makefile` in the root directory that includes common commands. The scaffolding tool has created a generic makefile that you will be able to use:

> _NOTE_: The below Makefile contains some of same commands as the Cosmos SDK and Tendermint Makefiles.

```makefile
PACKAGES=$(shell go list ./... | grep -v '/simulation')

VERSION := $(shell echo $(shell git describe --tags) | sed 's/^v//')
COMMIT := $(shell git log -1 --format='%H')

ldflags = -X github.com/cosmos/cosmos-sdk/version.Name=NameService \
	-X github.com/cosmos/cosmos-sdk/version.ServerName=nsd \
	-X github.com/cosmos/cosmos-sdk/version.ClientName=nscli \
	-X github.com/cosmos/cosmos-sdk/version.Version=$(VERSION) \
	-X github.com/cosmos/cosmos-sdk/version.Commit=$(COMMIT) 

BUILD_FLAGS := -ldflags '$(ldflags)'

include Makefile.ledger
all: install

install: go.sum
		@echo "--> Installing nsd & nscli"
		@go install -mod=readonly $(BUILD_FLAGS) ./cmd/nsd
		@go install -mod=readonly $(BUILD_FLAGS) ./cmd/nscli

go.sum: go.mod
		@echo "--> Ensure dependencies have not been modified"
		GO111MODULE=on go mod verify

test:
	@go test -mod=readonly $(PACKAGES)
```

How about including Ledger Nano S support? This requires a few small changes:

* Create a file `Makefile.ledger` with the following content:

```makefile
LEDGER_ENABLED ?= true

build_tags =
ifeq ($(LEDGER_ENABLED),true)
  ifeq ($(OS),Windows_NT)
    GCCEXE = $(shell where gcc.exe 2> NUL)
    ifeq ($(GCCEXE),)
      $(error gcc.exe not installed for ledger support, please install or set LEDGER_ENABLED=false)
    else
      build_tags += ledger
    endif
  else
    UNAME_S = $(shell uname -s)
    ifeq ($(UNAME_S),OpenBSD)
      $(warning OpenBSD detected, disabling ledger support (https://github.com/cosmos/cosmos-sdk/issues/1988))
    else
      GCC = $(shell command -v gcc 2> /dev/null)
      ifeq ($(GCC),)
        $(error gcc not installed for ledger support, please install or set LEDGER_ENABLED=false)
      else
        build_tags += ledger
      endif
    endif
  endif
endif
```

* Add `include Makefile.ledger` at the beginning of the Makefile:

```makefile
LEDGER_ENABLED ?= true

build_tags =
ifeq ($(LEDGER_ENABLED),true)
  ifeq ($(OS),Windows_NT)
    GCCEXE = $(shell where gcc.exe 2> NUL)
    ifeq ($(GCCEXE),)
      $(error gcc.exe not installed for ledger support, please install or set LEDGER_ENABLED=false)
    else
      build_tags += ledger
    endif
  else
    UNAME_S = $(shell uname -s)
    ifeq ($(UNAME_S),OpenBSD)
      $(warning OpenBSD detected, disabling ledger support (https://github.com/cosmos/cosmos-sdk/issues/1988))
    else
      GCC = $(shell command -v gcc 2> /dev/null)
      ifeq ($(GCC),)
        $(error gcc not installed for ledger support, please install or set LEDGER_ENABLED=false)
      else
        build_tags += ledger
      endif
    endif
  endif
endif
```

**`go.mod`**

Golang has a few dependency management tools. In this tutorial you will be using [`Go Modules`](https://github.com/golang/go/wiki/Modules). `Go Modules` uses a `go.mod` file in the root of the repository to define what dependencies the application needs. Cosmos SDK apps currently depend on specific versions of some libraries. The below manifest contains all the necessary versions. To get started replace the contents of the `./go.mod` file with the `constraints` and `overrides` below:

> _NOTE_: If you are following along in your own repo you will need to change the module path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

* You will have to run `go get ./...` to get all the modules the application is using. This command will get the dependency version stated in the `go.mod` file.
* If you would like to use a specific version of a dependency then you have to run `go get github.com/<github_org>/<repo_name>@<version>`

# Building the app 

Install the app into your $GOBIN

```text
make install
```

Now you should be able to run the following commands:

```text
nsd help
nscli help
```

Congratulations, you have finished your nameservice application! Try running and interacting with it!

## Run REST routes

Now that you tested your CLI queries and transactions, time to test same things in the REST server. Leave the `nsd` that you had running earlier and start by gathering your addresses:

```text
nscli keys show jack --address
nscli keys show alice --address
```

Now its time to start the `rest-server` in another terminal window:

```text
nscli rest-server --chain-id namechain --trust-node
```

Then you can construct and run the following queries:

> NOTE: Be sure to substitute your password and buyer/owner addresses for the ones listed below!

Get the sequence and account numbers for jack to construct the below requests:

```text
curl -s http://localhost:1317/auth/accounts/$(nscli keys show jack -a)
```

This should return similar output:

```text
{"type":"auth/Account","value":{"address":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","coins":[{"denom":"jackCoin","amount":"1000"},{"denom":"nametoken","amount":"1010"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"A9YxyEbSWzLr+IdK/PuMUYmYToKYQ3P/pM8SI1Bxx3wu"},"account_number":"0","sequence":"1"}}
```

Get the sequence and account numbers for alice to construct the below requests

```text
curl -s http://localhost:1317/auth/accounts/$(nscli keys show alice -a)
```

This should return similar output:

```text
{"type":"auth/Account","value":{"address":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl","coins":[{"denom":"aliceCoin","amount":"1000"},{"denom":"nametoken","amount":"980"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"Avc7qwecLHz5qb1EKDuSTLJfVOjBQezk0KSPDNybLONJ"},"account_number":"1","sequence":"2"}}
```

Buy another name for jack, first create the raw transaction

> **NOTE**: Be sure to specialize this request for your specific environment, also the "buyer" and "from" should be the same address

```text
curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"5nametoken","buyer":"'$(nscli keys show jack -a)'"}' > unsignedTx.json
```

Then sign this transaction. In a real environment the raw transaction should be signed on the client side. Also the sequence needs to be adjusted, depending on what the query of alice's account has shown.

```text
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 1 --account-number 0 > signedTx.json
```

Finally, broadcast the signed transaction:

```text
nscli tx broadcast signedTx.json
```

This should return similar output:

```text
{ "height": "266", "txhash": "C041AF0CE32FBAE5A4DD6545E4B1F2CB786879F75E2D62C79D690DAE163470BC", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted":"200000", "gas_used": "41510", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}
```

Set the data for that name that jack just bought. \
NOTE: Be sure to specialize this request for your specific environment, also the "owner" and "from" should be the same address

```text
curl -XPUT -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","value":"8.8.4.4","owner":"'$(nscli keys show jack -a)'"}' > unsignedTx.json
```

This should return similar output:

```text
{"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"1352","tags":[{"key":"YWN0aW9u","value":"c2V0X25hbWU="}]},"hash":"B4DF0105D57380D60524664A2E818428321A0DCA1B6B2F091FB3BEC54D68FAD7","height":"26"}
```

Again, we need to sign and broadcast:

```text
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 2 --account-number 0 > signedTx.json
nscli tx broadcast signedTx.json
```

Query the value for the name jack just set (should be 8.8.4.4):

```text
curl -s http://localhost:1317/nameservice/names/jack1.id
```

Query whois for the name jack just bought:

```
curl -s http://localhost:1317/nameservice/names/jack1.id/whois
```

This should return similar output:

```text
{"value":"8.8.8.8","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","price":[{"denom":"STAKE","amount":"10"}]}
```

Alice buys a name from jack:

```text
curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"10nametoken","buyer":"'$(nscli keys show alice -a)'"}' > unsignedTx.json
```

Again, we need to sign and broadcast. The account number has changed to 1 and the sequence is now 2, according to the query of alice's account:

```text
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 2 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json
```

This should return similar output:

```text
{ "height": "1515", "txhash": "C9DCC423E10E7E5E40A549057A4AA060DA6D6A885A394F6ED5C0E40AEE984A77", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted": "200000", "gas_used": "42375", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}
```

Now, Alice no longer needs the name she bought from jack and hence deletes it. Only the owner can delete the name. Since she is one, she can delete the name she bought from jack:

```text
curl -XDELETE -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","owner":"'$(nscli keys show alice -a)'"}' > unsignedTx.json
```

And a final time sign and broadcast. The account number is still 1, but the sequence is changed to 3, according to the query of alice's account:

```text
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 3 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json
```

Query whois for the name Alice just deleted:

```text
curl -s http://localhost:1317/nameservice/names/jack1.id/whois
```

```text
{"value":"","owner":"","price":[{"denom":"STAKE","amount":"1"}]}
```

## Request Schemas

**POST /nameservice/names BuyName Request Body:**

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "string,not_req",
  },
  "name": "string",
  "amount": "string",
  "buyer": "string"
}
```

**PUT /nameservice/names SetName Request Body:**

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "value": "string",
  "owner": "string"
}
```

**DELETE /nameservice/names DeleteName Request Body:**

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "owner": "string"
}

```

# Conclusion

Congratulations, you now have a working nameservice app running on Cosmos!


[The original tutorial can be found in the Cosmos SDK documentation here](https://tutorials.cosmos.network/scavenge/tutorial/01-background.html).

# Background

The goal of this session is to get you thinking about what is possible when developing applications that have access to **digital scarcity as a primitive**. The easiest way to think of scarcity is as money; If money grew on trees it would stop being _scarce_ and stop having value. We have a long history of software which deals with money, but it's never been a first class citizen in the programming environment. Instead, money has always been represented as a number or a float, and it has been up to a third party merchant service or some other process of exchange where the _representation_ of money is swapped for actual cash. If money were a primitive in a software environment, it would allow for **real economies to exist within games and applications**, taking one step further in erasing the line between games, life and play.

We will be working today with a Golang framework called the [Cosmos SDK](https://github.com/cosmos/cosmos-sdk). This framework makes it easy to build **deterministic state machines**. A state machine is simply an application that has a state and explicit functions for updating that state. You can think of a light bulb and a light switch as a kind of state machine: the state of the "application" is either `light on` or `light off`. There is one function in this state machine: `flip switch`. Every time you trigger `flip switch` the state of the application goes from `light on` to `light off` or vice versa. Simple, right?![](https://tutorials.cosmos.network/assets/img/light-bulb.9cb453da.png)

A **deterministic** state machine is just a state machine in which an accumulation of actions, taken together and replayed, will have the same outcome. So if we were to take all the `switch on` and `switch off` actions of the entire month of January for some room and replay then in August, we should have the same final state of `light on` or `light off`. There's should be nothing about January or August that changes the outcome \(of course a _real_ room might not be deterministic if there were things like power shortages or maintenance that took place during those periods\).

What is nice about deterministic state machines is that you can track changes with **cryptographic hashes** of the state, just like version control systems like `git`. If there is agreement about the hash of a certain state, it is unnecessary to replay every action from genesis to ensure that two repos are in sync with each other. These properties are useful when dealing with software that is run by many different people in many different situations, just like git.

Another nice property of cryptographically hashing state is that it creates a system of **reliable dependencies**. I can build software that uses your library and reference a specific state in your software. That way if you change your code in a way that breaks my code, I don't have to use your new version but can continue to use the version that I reference. This same property of knowing exactly what the state of a system \(as well as all the ways that state can update\) makes it possible to have the necessary assurances that allow for digital scarcity within an application. _If I say there is only one of some thing within a state machine and you know that there is no way for that state machine to create more than one, you can rely on there always being only one._![](https://tutorials.cosmos.network/assets/img/git.a9b7c1ec.jpeg)

You might have guessed by now that what I'm really talking about are **Blockchains**. These are deterministic state machines which have very specific rules about how state is updated. They checkpoint state with cryptographic hashes and use asymmetric cryptography to handle **access control**. There are different ways that different Blockchains decide who can make a checkpoint of state. These entities can be called **Validators**. Some of them are chosen by an electricity intensive game called **proof-of-work** in tandem with something called the longest chain rule or **Nakamoto Consensus** on Blockchains like Bitcoin or Ethereum.

The state machine we are building will use an implementation of **proof-of-stake** called **Tendermint**, which is energy efficient and can consist of one or many validators, either trusted or byzantine. When building a system that handles _real_ scarcity, the integrity of that system becomes very important. One way to ensure that integrity is by sharing the responsibility of maintaining it with a large group of independently motivated participants as validators.

So, now that we know a little more about **why** we might build an app like this, let's dive into the game itself.

# The Game

The application we're building today can be used in many different ways but I'll be talking about it as **scavenger hunt** game. Scavenger hunts are all about someone setting up tasks or questions that challenge a participant to find solutions which come with some sort of a prize. The basic mechanics of the game are as follows:

* **Anyone can post a question with an encrypted answer.**
* **This question comes paired with a bounty of coins.**
* **Anyone can post an answer to this question, if they get it right, they receive the bounty of coins.**

Something to note here is that when dealing with a public network with latency, it is possible that something like a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) could take place. Instead of pretending to be one of the parties, an attacker would take the sensitive information from one party and use it for their own benefit. This is actually called [Front Running](https://en.wikipedia.org/wiki/Front_running) and happens as follows:

1. You post the answer to some question with a bounty attacked to it.
2. Someone else sees you posting the answer and posts it themselves right before you.
3. Since they posted the answer first, they receive the reward instead of you.

To prevent Front-Running, we will implement a **commit-reveal** scheme. A commit-reveal scheme converts a single exploitable interaction and turns it into two safe interactions.

**The first interaction is the commit**. This is where you "commit" to posting an answer in a follow-up interaction. This commit consists of a cryptographic hash of your name combined with the answer that you think is correct. The app saves that value which is a claim that you know the answer but that it hasn't been confirmed whether the answer is correct.

**The next interaction is the reveal**. This is where you post the answer in plaintext along with your name. The application will take your answer and your name and cryptographically hash them. If the result matches what you previously submitted during the commit stage, then it will be proof that it is in fact you who knows the answer, and not someone who is just front-running you.

![](https://tutorials.cosmos.network/assets/img/front-run.fe321a27.jpg)

A system like this could be used in tandem with any kind of gaming platform in a **trustless** way. Imagine you were playing the legend of Zelda and the game was compiled with all the answers to different scavenger hunts already included. When you beat a level the game could reveal the secret answer. Then either explicitly or behind the scenes, this answer could be combined with your name, hashed, submitted and subsequently revealed. Your name would be rewarded and you would have more points in the game.

Another way of achieving this would be to have an Access Control List where there was an admin account that the video game company controlled. This admin account could confirm that you beat the level and then give you points. The problem with this is that it creates a \***single point of failure** and a single target for trying to attack the system. If there is one key that rules the castle then the whole system is broken if that key is compromised. It also creates a problem with coordination if that Admin account has to be online all the time in order for players to get their points. If you use a commit reveal system then you have a more trustless architecture where you don't need permission to play. This design decision has benefits and drawbacks, but paired with a careful implementation it can allow your game to scale without a single bottle neck or point of failure.

Now that we know what we're building we can get started.

# Scaffold

We'll be using a tool called [scaffold](https://github.com/cosmos/scaffold) to help us spin up a boilerplate app quickly. To use `scaffold` first clone and install it on your local machine:

```text
git clone git@github.com:cosmos/scaffold.git
cd scaffold
make tools
make install
scaffold --help
```

Afterwards, you should see the following help screen displayed:

```text
This CLI helps in scaffolding out CosmosSDK based applications

Usage:
  scaffold [command]

Available Commands:
  app         Generates an empty application boilerplate
  help        Help about any command
  module      Generate an empty module for use in the Cosmos-SDK
  tutorial    Generates one of the tutorial apps, currently either the 'nameservice' or 'hellochain'

Flags:
  -c, --config string        config file (default is $HOME/.scaffold.yaml)
  -h, --help                 help for scaffold
  -o, --output-path string   Path to output
  -t, --toggle               Help message for toggle

Use "scaffold [command] --help" for more information about a command.
```

Now that you have the `scaffold` command available try looking at the help screen of the `app` command by typing `scaffold app --help`.

```text
Generates an empty application boilerplate

Usage:
  scaffold app [lvl] [user] [repo] [flags]

Flags:
  -h, --help   help for app

Global Flags:
  -c, --config string        config file (default is $HOME/.scaffold.yaml)
  -o, --output-path string   Path to output
```

We will use this command to generate a basic boilerplate application. First create a new working directory on your machine that you can use to start a new project. You might want to do this in your home directory by using `cd ~`. It doesn't really matter where but you probably don't want to stay inside of your `scaffold` directory.

When it comes to starting your project with the `scaffold app` command, the parameter `lvl` should be filled with `lvl-1` \(which is currently the only lvl available\). You should use your own github username for `user` and come up with a name for `repo`. I will be using `scavenge` as the repo name for this tutorial. Using my own github handle \(`okwme`\) the final command should look like:

```text
scaffold app lvl-1 okwme scavenge
```

This should generate a folder structure inside of a directory called `scavenge` of your current working directory. Now that we have an app boilerplate we want to add some custom functionality to it and build our scavenge module. Change into the modules directory with `cd scavenge/x`. Now you can run the `module` command of the `scaffold` tool, but first check out the help screen of it with `scaffold module --help`:

```text
Generate an empty module for use in the Cosmos-SDK

Usage:
  scaffold module [user] [repo] [moduleName] [flags]

Flags:
  -h, --help   help for module

Global Flags:
  -c, --config string        config file (default is $HOME/.scaffold.yaml)
  -o, --output-path string   Path to output
```

Similarly, it asks for your github username as `user` and the name repository name as `repo`. It also asks for the name you'd like to give to this new module. I will use the name `scavenge` for the module as well.

```text
scaffold module okwme scavenge scavenge
```

Now that we have generated a boilerplate application with a boilerplate module, our next step will be to define our Messages.

# Messages 

Messages are a great place to start when building a module because they define the actions that your application can make. Think of all the scenarios where a user would be able to update the state of the application in any way. These should be boiled down into basic interactions, similar to **CRUD** \(Create, Read, Update, Delete\).

Let's start with **Create**

## MsgCreateScavenge

Messages are `types` which live inside the `./x/scavenge/types/` directory. There is already a `msg.go` file but we will make a new file for each Message type. We can use `msg.go` as a starting point by renaming it to `MsgCreateScavenge.go` like this (ssuming your current working directory is the root of your application):

```text
mv ./x/scavenge/types/msg.go  ./x/scavenge/types/MsgCreateScavenge.go
```

Inside this new file we will uncomment and follow the instructions of renaming variables until it looks as follows:

```go
package types

import (
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgCreateScavenge
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgCreateScavenge{}

// MsgCreateScavenge - struct for unjailing jailed validator
type MsgCreateScavenge struct {
	Creator      sdk.AccAddress `json:"creator" yaml:"creator"`           // address of the scavenger creator
	Description  string         `json:"description" yaml:"description"`   // description of the scavenge
	SolutionHash string         `json:"solutionHash" yaml:"solutionHash"` // solution hash of the scavenge
	Reward       sdk.Coins      `json:"reward" yaml:"reward"`             // reward of the scavenger
}

// NewMsgCreateScavenge creates a new MsgCreateScavenge instance
func NewMsgCreateScavenge(creator sdk.AccAddress, description, solutionHash string, reward sdk.Coins) MsgCreateScavenge {
	return MsgCreateScavenge{
		Creator:      creator,
		Description:  description,
		SolutionHash: solutionHash,
		Reward:       reward,
	}
}

// CreateScavengeConst is CreateScavenge Constant
const CreateScavengeConst = "CreateScavenge"

// nolint
func (msg MsgCreateScavenge) Route() string { return RouterKey }
func (msg MsgCreateScavenge) Type() string  { return CreateScavengeConst }
func (msg MsgCreateScavenge) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{sdk.AccAddress(msg.Creator)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgCreateScavenge) GetSignBytes() []byte {
	bz := ModuleCdc.MustMarshalJSON(msg)
	return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgCreateScavenge) ValidateBasic() error {
	if msg.Creator.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
	}
	if msg.SolutionHash == "" {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
	}
	return nil
}
```

Notice that all Messages in the app need to follow the `sdk.Msg` interface. The Message `struct` contains all the necessary information when creating a new scavenge:

* `Creator` - Who created it. This uses the `sdk.AccAddress` type which represents an account in the app controlled by public key cryptograhy.
* `Description` - What is the question to be solved or description of the challenge.
* `SolutionHash` - The scrambled solution.
* `Reward` - This is the bounty that is awarded to whoever submits the answer first.

The `Msg` interface requires some other methods be set, like validating the content of the `struct`, and confirming the msg was signed and submitted by the Creator.

Now that one can create a scavenge the only other essential action is to be able to solve it. This should be broken into two separate actions as described before: `MsgCommitSolution` and `MsgRevealSolution`.

## MsgCommitSolution 

This message type should live in `./x/scavenge/types/MsgCommitSolution.go` and look like:

```go
package types

import (
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgCommitSolution
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgCommitSolution{}

// MsgCommitSolution - struct for unjailing jailed validator
type MsgCommitSolution struct {
	Scavenger             sdk.AccAddress `json:"scavenger" yaml:"scavenger"`                         // address of the scavenger
	SolutionHash          string         `json:"solutionhash" yaml:"solutionhash"`                   // solutionhash of the scavenge
	SolutionScavengerHash string         `json:"solutionScavengerHash" yaml:"solutionScavengerHash"` // solution hash of the scavenge
}

// NewMsgCommitSolution creates a new MsgCommitSolution instance
func NewMsgCommitSolution(scavenger sdk.AccAddress, solutionHash string, solutionScavengerHash string) MsgCommitSolution {
	return MsgCommitSolution{
		Scavenger:             scavenger,
		SolutionHash:          solutionHash,
		SolutionScavengerHash: solutionScavengerHash,
	}
}

// CommitSolutionConst is CommitSolution Constant
const CommitSolutionConst = "CommitSolution"

// nolint
func (msg MsgCommitSolution) Route() string { return RouterKey }
func (msg MsgCommitSolution) Type() string  { return CommitSolutionConst }
func (msg MsgCommitSolution) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{sdk.AccAddress(msg.Scavenger)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgCommitSolution) GetSignBytes() []byte {
	bz := ModuleCdc.MustMarshalJSON(msg)
	return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgCommitSolution) ValidateBasic() error {
	if msg.Scavenger.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
	}
	if msg.SolutionHash == "" {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionHash can't be empty")
	}
	if msg.SolutionScavengerHash == "" {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
	}
	return nil
}
```

The Message `struct` contains all the necessary information when revealing a solution:

* `Scavenger` - Who is revealing the solution.
* `SolutionHash` - The scrambled solution \(hash\).
* `SolutionScavengerHash` - This is the hash of the combination of the solution and the person who solved it.

This message also fulfils the `sdk.Msg` interface.

## MsgRevealSolution 

This message type should live in `./x/scavenge/types/MsgRevealSolution.go` and look like:

```go
package types

import (
	"crypto/sha256"
	"encoding/hex"
	"fmt"

	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// MsgRevealSolution
// ------------------------------------------------------------------------------
var _ sdk.Msg = &MsgRevealSolution{}

// MsgRevealSolution - struct for unjailing jailed validator
type MsgRevealSolution struct {
	Scavenger    sdk.AccAddress `json:"scavenger" yaml:"scavenger"`       // address of the scavenger scavenger
	SolutionHash string         `json:"solutionHash" yaml:"solutionHash"` // SolutionHash of the scavenge
	Solution     string         `json:"solution" yaml:"solution"`         // solution of the scavenge
}

// NewMsgRevealSolution creates a new MsgRevealSolution instance
func NewMsgRevealSolution(scavenger sdk.AccAddress, solution string) MsgRevealSolution {

	var solutionHash = sha256.Sum256([]byte(solution))
	var solutionHashString = hex.EncodeToString(solutionHash[:])

	return MsgRevealSolution{
		Scavenger:    scavenger,
		SolutionHash: solutionHashString,
		Solution:     solution,
	}
}

// RevealSolutionConst is RevealSolution Constant
const RevealSolutionConst = "RevealSolution"

// nolint
func (msg MsgRevealSolution) Route() string { return RouterKey }
func (msg MsgRevealSolution) Type() string  { return RevealSolutionConst }
func (msg MsgRevealSolution) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{sdk.AccAddress(msg.Scavenger)}
}

// GetSignBytes gets the bytes for the message signer to sign on
func (msg MsgRevealSolution) GetSignBytes() []byte {
	bz := ModuleCdc.MustMarshalJSON(msg)
	return sdk.MustSortJSON(bz)
}

// ValidateBasic validity check for the AnteHandler
func (msg MsgRevealSolution) ValidateBasic() error {
	if msg.Scavenger.Empty() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, "creator can't be empty")
	}
	if msg.SolutionHash == "" {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionScavengerHash can't be empty")
	}
	if msg.Solution == "" {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "solutionHash can't be empty")
	}

	var solutionHash = sha256.Sum256([]byte(msg.Solution))
	var solutionHashString = hex.EncodeToString(solutionHash[:])

	if msg.SolutionHash != solutionHashString {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, fmt.Sprintf("Hash of solution (%s) doesn't equal solutionHash (%s)", msg.SolutionHash, solutionHashString))
	}
	return nil
}
```

The Message `struct` contains all the necessary information when revealing a solution:

* `Scavenger` - Who is revealing the solution.
* `SolutionHash` - The scrambled solution.
* `Solution` - This is the plain text version of the solution.

This message also fulfils the `sdk.Msg` interface.

# Codec 

Once we have defined our messages, we need to describe to our encoder how they should be stored as bytes. To do this we edit the file located at `./x/scavenge/types/codec.go`. By describing our types as follows they will work with our encoding library:

```go
package types

import (
	"github.com/cosmos/cosmos-sdk/codec"
)

// RegisterCodec registers concrete types on codec
func RegisterCodec(cdc *codec.Codec) {
	cdc.RegisterConcrete(MsgCreateScavenge{}, "scavenge/CreateScavenge", nil)
	cdc.RegisterConcrete(MsgCommitSolution{}, "scavenge/CommitSolution", nil)
	cdc.RegisterConcrete(MsgRevealSolution{}, "scavenge/RevealSolution", nil)
}

// ModuleCdc defines the module codec
var ModuleCdc *codec.Codec

func init() {
	ModuleCdc = codec.New()
	RegisterCodec(ModuleCdc)
	codec.RegisterCrypto(ModuleCdc)
	ModuleCdc.Seal()
}
```

# Alias

Now that we have these new message types, we'd like to make sure other parts of the module can access them. To do so we use the `./x/scavenge/alias.go` file. This imports the types from the nested `types` directory and makes them accessible at the modules top level directory.

```go
package scavenge

import (
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/keeper"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

const (
	ModuleName        = types.ModuleName
	RouterKey         = types.RouterKey
	StoreKey          = types.StoreKey
	DefaultParamspace = types.DefaultParamspace
	// QueryParams       = types.QueryParams
	QuerierRoute = types.QuerierRoute
)

var (
	// functions aliases
	NewKeeper           = keeper.NewKeeper
	NewQuerier          = keeper.NewQuerier
	RegisterCodec       = types.RegisterCodec
	NewGenesisState     = types.NewGenesisState
	DefaultGenesisState = types.DefaultGenesisState
	ValidateGenesis     = types.ValidateGenesis

	// variable aliases
	ModuleCdc = types.ModuleCdc

	NewMsgCreateScavenge = types.NewMsgCreateScavenge
	NewMsgCommitSolution = types.NewMsgCommitSolution
	NewMsgRevealSolution = types.NewMsgRevealSolution
)

type (
	Keeper       = keeper.Keeper
	GenesisState = types.GenesisState
	Params       = types.Params

	MsgCreateScavenge = types.MsgCreateScavenge
	MsgCommitSolution = types.MsgCommitSolution
	MsgRevealSolution = types.MsgRevealSolution
)
```

It's great to have Messages, but we need somewhere to store the information they are sending. All persistent data related to this module should live in the module's `Keeper`.

Let's make a `Keeper` for our Scavenge Module next.

# Keeper 

After using the `scaffold` command you should have a boilerplate `Keeper` at `./x/scavenge/keeper/keeper.go`. It contains a basic keeper with references to basic functions like `Set`, `Get` and `Delete`.

Our keeper stores all our data for our module. Sometimes a module will import the keeper of another module. This will allow state to be shared and modified across modules. Since we are dealing with coins in our module as bounty rewards, we will need to access the `bank` module's keeper \(which we call `CoinKeeper`\). Look at our completed `Keeper` and you can see where the `bank` keeper is referenced and how `Set`, `Get` and `Delete` are expanded:

```go
package keeper

import (
	"fmt"

	"github.com/tendermint/tendermint/libs/log"

	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/x/bank"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

// Keeper of the scavenge store
type Keeper struct {
	CoinKeeper bank.Keeper
	storeKey   sdk.StoreKey
	cdc        *codec.Codec
}

// NewKeeper creates a scavenge keeper
func NewKeeper(coinKeeper bank.Keeper, cdc *codec.Codec, key sdk.StoreKey) Keeper {
	keeper := Keeper{
		CoinKeeper: coinKeeper,
		storeKey:   key,
		cdc:        cdc,
	}
	return keeper
}

// Logger returns a module-specific logger.
func (k Keeper) Logger(ctx sdk.Context) log.Logger {
	return ctx.Logger().With("module", fmt.Sprintf("x/%s", types.ModuleName))
}

// GetCommit returns the commit of a solution
func (k Keeper) GetCommit(ctx sdk.Context, solutionScavengerHash string) (types.Commit, error) {
	store := ctx.KVStore(k.storeKey)
	var commit types.Commit
	byteKey := []byte(types.CommitPrefix + solutionScavengerHash)
	err := k.cdc.UnmarshalBinaryLengthPrefixed(store.Get(byteKey), &commit)
	if err != nil {
		return commit, err
	}
	return commit, nil
}

// GetScavenge returns the scavenge information
func (k Keeper) GetScavenge(ctx sdk.Context, solutionHash string) (types.Scavenge, error) {
	store := ctx.KVStore(k.storeKey)
	var scavenge types.Scavenge
	byteKey := []byte(types.ScavengePrefix + solutionHash)
	err := k.cdc.UnmarshalBinaryLengthPrefixed(store.Get(byteKey), &scavenge)
	if err != nil {
		return scavenge, err
	}
	return scavenge, nil
}

// SetCommit sets a scavenge
func (k Keeper) SetCommit(ctx sdk.Context, commit types.Commit) {
	solutionScavengerHash := commit.SolutionScavengerHash
	store := ctx.KVStore(k.storeKey)
	bz := k.cdc.MustMarshalBinaryLengthPrefixed(commit)
	key := []byte(types.CommitPrefix + solutionScavengerHash)
	store.Set(key, bz)
}

// SetScavenge sets a scavenge
func (k Keeper) SetScavenge(ctx sdk.Context, scavenge types.Scavenge) {
	solutionHash := scavenge.SolutionHash
	store := ctx.KVStore(k.storeKey)
	bz := k.cdc.MustMarshalBinaryLengthPrefixed(scavenge)
	key := []byte(types.ScavengePrefix + solutionHash)
	store.Set(key, bz)
}

// DeleteScavenge deletes a scavenge
func (k Keeper) DeleteScavenge(ctx sdk.Context, solutionHash string) {
	store := ctx.KVStore(k.storeKey)
	store.Delete([]byte(solutionHash))
}

// GetScavengesIterator gets an iterator over all scavnges in which the keys are the solutionHashes and the values are the scavenges
func (k Keeper) GetScavengesIterator(ctx sdk.Context) sdk.Iterator {
	store := ctx.KVStore(k.storeKey)
	return sdk.KVStorePrefixIterator(store, []byte(types.ScavengePrefix))
}

// GetCommitsIterator gets an iterator over all commits in which the keys are the prefix and solutionHashes and the values are the scavenges
func (k Keeper) GetCommitsIterator(ctx sdk.Context) sdk.Iterator {
	store := ctx.KVStore(k.storeKey)
	return sdk.KVStorePrefixIterator(store, []byte(types.CommitPrefix))
}
```

## Commits and Scavenges 

You may notice reference to `types.Commit` and `types.Scavenge` throughout the `Keeper`. These are new structs defined in `./x/scavenge/types/types.go` that contain all necessary information about different scavenge challenges, and different commited solutions to those challenges. They appear similar to the `Msg` types we saw earlier because they contain similar information. You can create this file now and add the following:

```go
package types

import (
	"fmt"
	"strings"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

// Scavenge is the Scavenge struct
type Scavenge struct {
	Creator      sdk.AccAddress `json:"creator" yaml:"creator"`           // address of the scavenger creator
	Description  string         `json:"description" yaml:"description"`   // description of the scavenge
	SolutionHash string         `json:"solutionHash" yaml:"solutionHash"` // solution hash of the scavenge
	Reward       sdk.Coins      `json:"reward" yaml:"reward"`             // reward of the scavenger
	Solution     string         `json:"solution" yaml:"solution"`         // the solution to the scagenve
	Scavenger    sdk.AccAddress `json:"scavenger" yaml:"scavenger"`       // the scavenger who found the solution
}

// implement fmt.Stringer
func (s Scavenge) String() string {
	return strings.TrimSpace(fmt.Sprintf(`Creator: %s
	Description: %s
	SolutionHash: %s
	Reward: %s
	Solution: %s
	Scavenger: %s`,
		s.Creator,
		s.Description,
		s.SolutionHash,
		s.Reward,
		s.Solution,
		s.Scavenger,
	))
}

// Commit is the commit struct
type Commit struct {
	Scavenger             sdk.AccAddress `json:"scavenger" yaml:"scavenger"`                         // address of the scavenger scavenger
	SolutionHash          string         `json:"solutionHash" yaml:"solutionHash"`                   // SolutionHash of the scavenge
	SolutionScavengerHash string         `json:"solutionScavengerHash" yaml:"solutionScavengerHash"` // solution hash of the scavenge
}

// implement fmt.Stringer
func (c Commit) String() string {
	return strings.TrimSpace(fmt.Sprintf(`Scavenger: %s
	SolutionHash: %s
	SolutionScavengerHash: %s`,
		c.Scavenger,
		c.SolutionHash,
		c.SolutionScavengerHash,
	))
}
```

You can imagine that an unsolved `Scavenge` would contain a `nil` value for the fields `Solution` and `Scavenger` before they are solved. You might also notice that each type has the `String` method. This allows us to render the struct as a string for rendering.

## Prefixes

You may notice the use of `types.ScavengePrefix` and `types.CommitPrefix`. These are defined in a file called `./x/scavenge/types/key.go` and help us keep our `Keeper` organized. The `Keeper` is really just a key value store. That means that, similar to an `Object` in javascript, all values are referenced under a key. To access a value, you need to know the key under which it is stored. This is a bit like a unique identifier \(UID\).

When storing a `Scavenge` we use the key of the `SolutionHash` as a unique ID, for a `Commit` we use the key of the `SolutionScavengeHash`. However since we are storing these two data types in the same location, we may want to distinguish between the types of hashes we use as keys. We can do this by adding prefixes to the hashes that allow us to recognize which is which. For `Scavenge` we add the prefix `sk-` and for `Commit` we add the prefix `ck-`. You should add these to your `key.go` file so it looks as follows:

```go
package types

const (
	// ModuleName is the name of the module
	ModuleName = "scavenge"

	// StoreKey to be used when creating the KVStore
	StoreKey = ModuleName

	// RouterKey to be used for routing msgs
	RouterKey = ModuleName

	QuerierRoute = ModuleName

	ScavengePrefix = "sk-"
	CommitPrefix   = "ck-"
)
```

Copypackage types const \( // ModuleName is the name of the module ModuleName = "scavenge" // StoreKey to be used when creating the KVStore StoreKey = ModuleName // RouterKey to be used for routing msgs RouterKey = ModuleName QuerierRoute = ModuleName ScavengePrefix = "sk-" CommitPrefix = "ck-" \)

## Iterators 

Sometimes you will want to access a `Commit` or a `Scavenge` directly by their key. That's why we have the methods `GetCommit` and `GetScavenge`. However, sometimes you will want to get every `Scavenge` at once or every `Commit` at once. To do this we use an **Iterator** called `KVStorePrefixIterator`. This utility comes from the `sdk` and iterates over a key store. If you provide a prefix, it will only iterate over the keys that contain that prefix. Since we have prefixes defined for our `Scavenge` and our `Commit` we can use them here to only return our desired data types.

Now that you've seen the `Keeper` where every `Commit` and `Scavenge` are stored, we need to connect the messages to the this storage. This process is called _handling_ the messages and is done inside the `Handler`.

# Handler 

In order for a **Message** to reach a **Keeper**, it has to go through a **Handler**. This is where logic can be applied to either allow or deny a `Message` to succeed. It's also where logic as to exactly how the state should change within the Keeper should take place. If you're familiar with [Model View Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) \(MVC\) architecture, the `Keeper` is a bit like the **Model** and the `Handler` is a bit like the **Controller**. If you're familiar with [React/Redux](https://en.wikipedia.org/wiki/React_%28web_framework%29) or [Vue/Vuex](https://en.wikipedia.org/wiki/Vue.js) architecture, the `Keeper` is a bit like the **Reducer/Store** and the `Handler` is a bit like **Actions**.

Our Handler will go in `./x/scavenge/handler.go` and will follow the suggestions outlined in the boilerplate. We will create handler functions for each of our three `Message` types, `MsgCreateScavenge`, `MsgCommitSolution` and `MsgRevealSolution` until the file looks as follows:

```go
package scavenge

import (
	"crypto/sha256"
	"encoding/hex"
	"fmt"

	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
	"github.com/tendermint/tendermint/crypto"
)

// NewHandler creates an sdk.Handler for all the scavenge type messages
func NewHandler(k Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
		ctx = ctx.WithEventManager(sdk.NewEventManager())
		switch msg := msg.(type) {
		case MsgCreateScavenge:
			return handleMsgCreateScavenge(ctx, k, msg)
		case MsgCommitSolution:
			return handleMsgCommitSolution(ctx, k, msg)
		case MsgRevealSolution:
			return handleMsgRevealSolution(ctx, k, msg)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest,
				fmt.Sprintf("unrecognized %s message type: %T", types.ModuleName, msg))
		}
	}
}

// handleMsgCreateScavenge creates a new scavenge and moves the reward into escrow
func handleMsgCreateScavenge(ctx sdk.Context, k Keeper, msg MsgCreateScavenge) (*sdk.Result, error) {
	var scavenge = types.Scavenge{
		Creator:      msg.Creator,
		Description:  msg.Description,
		SolutionHash: msg.SolutionHash,
		Reward:       msg.Reward,
	}
	_, err := k.GetScavenge(ctx, scavenge.SolutionHash)
	if err == nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Scavenge with that solution hash already exists")
	}
	moduleAcct := sdk.AccAddress(crypto.AddressHash([]byte(types.ModuleName)))
	sdkError := k.CoinKeeper.SendCoins(ctx, scavenge.Creator, moduleAcct, scavenge.Reward)
	if sdkError != nil {
		return nil, sdkError
	}
	k.SetScavenge(ctx, scavenge)
	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			sdk.EventTypeMessage,
			sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
			sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeCreateScavenge),
			sdk.NewAttribute(sdk.AttributeKeySender, msg.Creator.String()),
			sdk.NewAttribute(types.AttributeDescription, msg.Description),
			sdk.NewAttribute(types.AttributeSolutionHash, msg.SolutionHash),
			sdk.NewAttribute(types.AttributeReward, msg.Reward.String()),
		),
	)
	return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}

func handleMsgCommitSolution(ctx sdk.Context, k Keeper, msg MsgCommitSolution) (*sdk.Result, error) {
	var commit = types.Commit{
		Scavenger:             msg.Scavenger,
		SolutionHash:          msg.SolutionHash,
		SolutionScavengerHash: msg.SolutionScavengerHash,
	}
	_, err := k.GetCommit(ctx, commit.SolutionScavengerHash)
	// should produce an error when commit is not found
	if err == nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Commit with that hash already exists")
	}
	k.SetCommit(ctx, commit)
	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			sdk.EventTypeMessage,
			sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
			sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeCommitSolution),
			sdk.NewAttribute(sdk.AttributeKeySender, msg.Scavenger.String()),
			sdk.NewAttribute(types.AttributeSolutionHash, msg.SolutionHash),
			sdk.NewAttribute(types.AttributeSolutionScavengerHash, msg.SolutionScavengerHash),
		),
	)
	return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}

func handleMsgRevealSolution(ctx sdk.Context, k Keeper, msg MsgRevealSolution) (*sdk.Result, error) {
	var solutionScavengerBytes = []byte(msg.Solution + msg.Scavenger.String())
	var solutionScavengerHash = sha256.Sum256(solutionScavengerBytes)
	var solutionScavengerHashString = hex.EncodeToString(solutionScavengerHash[:])
	_, err := k.GetCommit(ctx, solutionScavengerHashString)
	if err != nil {
		return nil, sdkerrors.Wrap(err, "Commit with that hash doesn't exists")
	}

	var solutionHash = sha256.Sum256([]byte(msg.Solution))
	var solutionHashString = hex.EncodeToString(solutionHash[:])
	var scavenge types.Scavenge
	scavenge, err = k.GetScavenge(ctx, solutionHashString)
	if err != nil {
		return nil, sdkerrors.Wrap(err, "Scavenge with that solution hash doesn't exists")
	}
	if scavenge.Scavenger != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Scavenge has already been solved")
	}
	scavenge.Scavenger = msg.Scavenger
	scavenge.Solution = msg.Solution

	moduleAcct := sdk.AccAddress(crypto.AddressHash([]byte(types.ModuleName)))
	sdkError := k.CoinKeeper.SendCoins(ctx, moduleAcct, scavenge.Scavenger, scavenge.Reward)
	if sdkError != nil {
		return nil, sdkError
	}
	k.SetScavenge(ctx, scavenge)
	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			sdk.EventTypeMessage,
			sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory),
			sdk.NewAttribute(sdk.AttributeKeyAction, types.EventTypeSolveScavenge),
			sdk.NewAttribute(sdk.AttributeKeySender, msg.Scavenger.String()),
			sdk.NewAttribute(types.AttributeSolutionHash, solutionHashString),
			sdk.NewAttribute(types.AttributeDescription, scavenge.Description),
			sdk.NewAttribute(types.AttributeSolution, msg.Solution),
			sdk.NewAttribute(types.AttributeScavenger, scavenge.Scavenger.String()),
			sdk.NewAttribute(types.AttributeReward, scavenge.Reward.String()),
		),
	)
	return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}
```

## moduleAcct 

You might notice the use of `moduleAcct` within the `handleMsgCreateScavenge` and `handleMsgRevealSolution` handler functions. This account is not controlled by a public key pair, but is a reference to an account that is owned by this actual module. It is used to hold the bounty reward that is attached to a scavenge until that scavenge has been solved, at which point the bounty is paid to the account who solved the scavenge.

# Events 

At the end of each handler is an `EventManager` which will create logs within the transaction that reveals information about what occurred during the handling of this message. This is useful for client side software that wants to know exactly what happened as a result of this state transition. These Events use a series of pre-defined types that can be found in `./x/scavenge/types/events.go` and look as follows:

```go
package types

// scavenge module event types
const (
	EventTypeCreateScavenge = "CreateScavenge"
	EventTypeCommitSolution = "CommitSolution"
	EventTypeSolveScavenge  = "SolveScavenge"

	AttributeDescription           = "description"
	AttributeSolution              = "solution"
	AttributeSolutionHash          = "solutionHash"
	AttributeReward                = "reward"
	AttributeScavenger             = "scavenger"
	AttributeSolutionScavengerHash = "solutionScavengerHash"

	AttributeValueCategory = ModuleName
)
```

Now that we have all the necessary pieces for updating state \(`Message`, `Handler`, `Keeper`\) we might want to consider ways in which we can _query_ state. This is typically done via a REST endpoint and/or a CLI. Both of those clients interact with part of the app which queries state, called the `Querier`.

# Querier 

In order to query the data of our app we need to make it accessible using our `Querier`. This piece of the app works in tandem with the `Keeper` to access state and return it. The `Querier` is defined in `./x/scavenge/keeper/querier.go`. Our `scaffold` tool starts us out with some suggestions on how it should look, and similar to our `Handler` we want to handle different queried routes. You could make many different routes within the `Querier` for many different types of queries, but we will just make three:

* `listScavenges` will list all scavenges
* `getScavenge` will get a single scavenge by `solutionHash`
* `getCommit` will get a single commit by `solutionScavengerHash`

Combined into a switch statement and with each of the functions fleshed out it should look as follows:

```go
package keeper

import (
	abci "github.com/tendermint/tendermint/abci/types"

	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

// NewQuerier creates a new querier for scavenge clients.
func NewQuerier(k Keeper) sdk.Querier {
	return func(ctx sdk.Context, path []string, req abci.RequestQuery) ([]byte, error) {
		switch path[0] {
		case types.QueryListScavenges:
			return listScavenges(ctx, k)
		case types.QueryGetScavenge:
			return getScavenge(ctx, path[1:], k)
		case types.QueryCommit:
			return getCommit(ctx, path[1:], k)
		default:
			return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "unknown scavenge query endpoint")
		}
	}
}

// RemovePrefixFromHash removes the prefix from the key
func RemovePrefixFromHash(key []byte, prefix []byte) (hash []byte) {
	hash = key[len(prefix):]
	return hash
}

func listScavenges(ctx sdk.Context, k Keeper) ([]byte, error) {
	var scavengeList types.QueryResScavenges

	iterator := k.GetScavengesIterator(ctx)

	for ; iterator.Valid(); iterator.Next() {
		scavengeHash := RemovePrefixFromHash(iterator.Key(), []byte(types.ScavengePrefix))
		scavengeList = append(scavengeList, string(scavengeHash))
	}

	res, err := codec.MarshalJSONIndent(k.cdc, scavengeList)
	if err != nil {
		return res, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}

	return res, nil
}

func getScavenge(ctx sdk.Context, path []string, k Keeper) (res []byte, sdkError error) {
	solutionHash := path[0]
	scavenge, err := k.GetScavenge(ctx, solutionHash)
	if err != nil {
		return nil, err
	}

	res, err = codec.MarshalJSONIndent(k.cdc, scavenge)
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}

	return res, nil
}

func getCommit(ctx sdk.Context, path []string, k Keeper) (res []byte, sdkError error) {
	solutionScavengerHash := path[0]
	commit, err := k.GetCommit(ctx, solutionScavengerHash)
	if err != nil {
		return nil, err
	}
	res, err = codec.MarshalJSONIndent(k.cdc, commit)
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
	}
	return res, nil
}
```

## Types 

You may notice that we use three different imported types on our initial switch statement. These are defined within our `./x/scavenge/types/querier.go` file as simple strings. That file should look like the following:

```go
package types

import "strings"

// query endpoints supported by the nameservice Querier
const (
	QueryListScavenges = "list"
	QueryGetScavenge   = "get"
	QueryCommit        = "commit"
)

// // QueryResResolve Queries Result Payload for a resolve query
// type QueryResResolve struct {
// 	Value string `json:"value"`
// }

// // implement fmt.Stringer
// func (r QueryResResolve) String() string {
// 	return r.Value
// }

// QueryResScavenges Queries Result Payload for a names query
type QueryResScavenges []string

// implement fmt.Stringer
func (n QueryResScavenges) String() string {
	return strings.Join(n[:], "\n")
}
```

Our queries are rather simple since we've already outfitted our `Keeper` with all the necessary functions to access state. You can see the iterator being used here as well.

Now that we have all of the basic actions of our module created, we want to make them accessible. We can do this with a CLI client and a REST client. For this tutorial we will just be creating a CLI client. If you are interested in what goes into making a REST client, check out the [Nameservice Tutorial](https://tutorials.cosmos.network/nameservice/tutorial/00-intro.html).

Let's take a look at what goes into making a CLI.

# CLI 

A Command Line Interface \(CLI\) will help us interact with our app once it is running on a machine somewhere. Each Module has it's own namespace within the CLI that gives it the ability to create and sign Messages destined to be handled by that module. It also comes with the ability to query the state of that module. When combined with the rest of the app, the CLI will let you do things like generate keys for a new account or check the status of an interaction you already had with the application.

The CLI for our module is broken into two files called `tx.go` and `query.go` which are located in `./x/scavenge/client/cli/`. One file is for making transactions that contain messages which will ultimately update our state. The other is for making queries which will give us the ability to read information from our state. Both files utilize the [Cobra](https://github.com/spf13/cobra) library.

## tx.go 

The `tx.go` file contains `GetTxCmd` which is a standard method within the Cosmos SDK. It is referenced later in the `module.go` file which describes exactly which attributes a modules has. This makes it easier to incorporate different modules for different reasons at the level of the actual application. After all, we are focusing on a module at this point, but later we will create an application that utilizes this module as well as other modules which are already available within the Cosmos SDK.

Inside `GetTxCmd` we create a new module-specific command and call is `scavenge`. Within this command we add a sub-command for each Message type we've defined:

* `GetCmdCreateScavenge`
* `GetCmdCommitSolution`
* `GetCmdRevealSolution`

Each function takes parameters from the **Cobra** CLI tool to create a new msg, sign it and submit it to the application to be processed. These functions should go into the `tx.go` file and look as follows:

```go
package cli

import (
	"bufio"
	"crypto/sha256"
	"encoding/hex"
	"fmt"

	"github.com/spf13/cobra"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/x/auth"
	"github.com/cosmos/cosmos-sdk/x/auth/client/utils"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

// GetTxCmd returns the transaction commands for this module
func GetTxCmd(cdc *codec.Codec) *cobra.Command {
	scavengeTxCmd := &cobra.Command{
		Use:                        types.ModuleName,
		Short:                      fmt.Sprintf("%s transactions subcommands", types.ModuleName),
		DisableFlagParsing:         true,
		SuggestionsMinimumDistance: 2,
		RunE:                       client.ValidateCmd,
	}

	scavengeTxCmd.AddCommand(flags.PostCommands(
		GetCmdCreateScavenge(cdc),
		GetCmdCommitSolution(cdc),
		GetCmdRevealSolution(cdc),
	)...)

	return scavengeTxCmd
}

func GetCmdCreateScavenge(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "createScavenge [reward] [solution] [description]",
		Short: "Creates a new scavenge with a reward",
		Args:  cobra.ExactArgs(3), // Does your request require arguments
		RunE: func(cmd *cobra.Command, args []string) error {

			cliCtx := context.NewCLIContext().WithCodec(cdc)
			inBuf := bufio.NewReader(cmd.InOrStdin())
			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			reward, err := sdk.ParseCoins(args[0])
			if err != nil {
				return err
			}

			var solution = args[1]
			var solutionHash = sha256.Sum256([]byte(solution))
			var solutionHashString = hex.EncodeToString(solutionHash[:])

			msg := types.NewMsgCreateScavenge(cliCtx.GetFromAddress(), args[2], solutionHashString, reward)
			err = msg.ValidateBasic()
			if err != nil {
				return err
			}

			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}

func GetCmdCommitSolution(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "commitSolution [solution]",
		Short: "Commits a solution for scavenge",
		Args:  cobra.ExactArgs(1), // Does your request require arguments
		RunE: func(cmd *cobra.Command, args []string) error {

			cliCtx := context.NewCLIContext().WithCodec(cdc)
			inBuf := bufio.NewReader(cmd.InOrStdin())
			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			var solution = args[0]
			var solutionHash = sha256.Sum256([]byte(solution))
			var solutionHashString = hex.EncodeToString(solutionHash[:])

			var scavenger = cliCtx.GetFromAddress().String()

			var solutionScavengerHash = sha256.Sum256([]byte(solution + scavenger))
			var solutionScavengerHashString = hex.EncodeToString(solutionScavengerHash[:])

			msg := types.NewMsgCommitSolution(cliCtx.GetFromAddress(), solutionHashString, solutionScavengerHashString)
			err := msg.ValidateBasic()
			if err != nil {
				return err
			}
			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}

func GetCmdRevealSolution(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "revealSolution [solution]",
		Short: "Reveals a solution for scavenge",
		Args:  cobra.ExactArgs(1), // Does your request require arguments
		RunE: func(cmd *cobra.Command, args []string) error {

			cliCtx := context.NewCLIContext().WithCodec(cdc)
			inBuf := bufio.NewReader(cmd.InOrStdin())
			txBldr := auth.NewTxBuilderFromCLI(inBuf).WithTxEncoder(utils.GetTxEncoder(cdc))

			var solution = args[0]

			msg := types.NewMsgRevealSolution(cliCtx.GetFromAddress(), solution)
			err := msg.ValidateBasic()
			if err != nil {
				return err
			}

			return utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})
		},
	}
}
```

## sha256

Note that this file makes use of the `sha256` library for hashing our plain text solutions into the scrambled hashes. This activity takes place on the client side so the solutions are never leaked to any public entity which might want to sneak a peak and steal the bounty reward associated with the scavenges. You can also notice that the hashes are converted into hexadecimal representation to make them easy to read as strings \(which is how they are ultimately stored in the keeper\).

## query.go 

The `query.go` file contains similar **Cobra** commands that reserve a new name space for referencing our `scavenge` module. Instead of creating and submitting messages however, the `query.go` file creates queries and returns the results in human readable form. The queries it handles are the same we defined in our `querier.go` file earlier:

* `GetCmdListScavenges`
* `GetCmdGetScavenge`
* `GetCmdGetCommit`

After defining these commands, your `query.go` file should look like:

```go
package cli

import (
	"crypto/sha256"
	"encoding/hex"
	"fmt"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/spf13/cobra"

	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

// GetQueryCmd returns the cli query commands for this module
func GetQueryCmd(queryRoute string, cdc *codec.Codec) *cobra.Command {
	// Group scavenge queries under a subcommand
	scavengeQueryCmd := &cobra.Command{
		Use:                        types.ModuleName,
		Short:                      fmt.Sprintf("Querying commands for the %s module", types.ModuleName),
		DisableFlagParsing:         true,
		SuggestionsMinimumDistance: 2,
		RunE:                       client.ValidateCmd,
	}

	scavengeQueryCmd.AddCommand(
		flags.GetCommands(
			GetCmdListScavenges(queryRoute, cdc),
			GetCmdGetScavenge(queryRoute, cdc),
			GetCmdGetCommit(queryRoute, cdc),
		)...,
	)

	return scavengeQueryCmd

}

func GetCmdListScavenges(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "list",
		Short: "list",
		// Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/"+types.QueryListScavenges, queryRoute), nil)
			if err != nil {
				fmt.Printf("could not get scavenges\n%s\n", err.Error())
				return nil
			}

			var out types.QueryResScavenges
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}
func GetCmdGetScavenge(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "get [solutionHash]",
		Short: "Query a scavenge by solutionHash",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			solutionHash := args[0]

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/%s/%s", queryRoute, types.QueryGetScavenge, solutionHash), nil)
			if err != nil {
				fmt.Printf("could not resolve scavenge %s \n%s\n", solutionHash, err.Error())

				return nil
			}

			var out types.Scavenge
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}
func GetCmdGetCommit(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "commited [solution] [scavenger]",
		Short: "Query a commit by solution and address of scavenger",
		Args:  cobra.ExactArgs(2),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)

			var solution = args[0]
			var solutionHash = sha256.Sum256([]byte(solution))
			var solutionHashString = hex.EncodeToString(solutionHash[:])

			var scavenger = args[1]

			var solutionScavengerHash = sha256.Sum256([]byte(solution + scavenger))
			var solutionScavengerHashString = hex.EncodeToString(solutionScavengerHash[:])

			res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/%s/%s", queryRoute, types.QueryCommit, solutionScavengerHashString), nil)
			if err != nil {
				fmt.Printf("could not resolve commit %s for scavenge %s \n%s\n", solutionScavengerHashString, solutionHashString, err.Error())
				return nil
			}

			var out types.Commit
			cdc.MustUnmarshalJSON(res, &out)
			return cliCtx.PrintOutput(out)
		},
	}
}
```

Notice that this file also makes use of the `sha256` library for converting plain text into hexadecimal hash strings.

While these are all the major moving pieces of a module \(`Message`, `Handler`, `Keeper`, `Querier` and `Client`\) there are some organizational tasks which we have yet to complete. The next step will be making sure that our module is completely configured in order to make it usable within any application.

# Module 

Our `scaffold` tool has done most of the work for us in generating our `module.go` file inside `./x/scavenge/`. One way that our module is different than the simplest form of a module, is that it uses it's own `Keeper` as well as the `Keeper` from the `bank` module. The only real changes needed are under the `AppModule` and `NewAppModule`, where the `bank.Keeper` needs to be added and referenced. The file should look as follows afterwards:

```go
package scavenge

import (
	"encoding/json"

	"github.com/gorilla/mux"
	"github.com/spf13/cobra"

	abci "github.com/tendermint/tendermint/abci/types"

	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/module"
	"github.com/cosmos/cosmos-sdk/x/bank"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/client/cli"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/client/rest"
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

var (
	_ module.AppModule      = AppModule{}
	_ module.AppModuleBasic = AppModuleBasic{}
)

// AppModuleBasic defines the basic application module used by the scavenge module.
type AppModuleBasic struct{}

var _ module.AppModuleBasic = AppModuleBasic{}

// Name returns the scavenge module's name.
func (AppModuleBasic) Name() string {
	return types.ModuleName
}

// RegisterCodec registers the scavenge module's types for the given codec.
func (AppModuleBasic) RegisterCodec(cdc *codec.Codec) {
	RegisterCodec(cdc)
}

// DefaultGenesis returns default genesis state as raw bytes for the scavenge
// module.
func (AppModuleBasic) DefaultGenesis() json.RawMessage {
	return ModuleCdc.MustMarshalJSON(DefaultGenesisState())
}

// ValidateGenesis performs genesis state validation for the scavenge module.
func (AppModuleBasic) ValidateGenesis(bz json.RawMessage) error {
	var data GenesisState
	err := ModuleCdc.UnmarshalJSON(bz, &data)
	if err != nil {
		return err
	}
	return ValidateGenesis(data)
}

// RegisterRESTRoutes registers the REST routes for the scavenge module.
func (AppModuleBasic) RegisterRESTRoutes(ctx context.CLIContext, rtr *mux.Router) {
	rest.RegisterRoutes(ctx, rtr)
}

// GetTxCmd returns the root tx command for the scavenge module.
func (AppModuleBasic) GetTxCmd(cdc *codec.Codec) *cobra.Command {
	return cli.GetTxCmd(cdc)
}

// GetQueryCmd returns no root query command for the scavenge module.
func (AppModuleBasic) GetQueryCmd(cdc *codec.Codec) *cobra.Command {
	return cli.GetQueryCmd(StoreKey, cdc)
}

//____________________________________________________________________________

// AppModule implements an application module for the scavenge module.
type AppModule struct {
	AppModuleBasic
	keeper     Keeper
	coinKeeper bank.Keeper
}

// NewAppModule creates a new AppModule object
func NewAppModule(k Keeper, bankKeeper bank.Keeper) AppModule {
	return AppModule{
		AppModuleBasic: AppModuleBasic{},
		keeper:         k,
		coinKeeper:     bankKeeper,
	}
}

// Name returns the scavenge module's name.
func (AppModule) Name() string {
	return ModuleName
}

// RegisterInvariants registers the scavenge module invariants.
func (am AppModule) RegisterInvariants(_ sdk.InvariantRegistry) {}

// Route returns the message routing key for the scavenge module.
func (AppModule) Route() string {
	return RouterKey
}

// NewHandler returns an sdk.Handler for the scavenge module.
func (am AppModule) NewHandler() sdk.Handler {
	return NewHandler(am.keeper)
}

// QuerierRoute returns the scavenge module's querier route name.
func (AppModule) QuerierRoute() string {
	return QuerierRoute
}

// NewQuerierHandler returns the scavenge module sdk.Querier.
func (am AppModule) NewQuerierHandler() sdk.Querier {
	return NewQuerier(am.keeper)
}

// InitGenesis performs genesis initialization for the scavenge module. It returns
// no validator updates.
func (am AppModule) InitGenesis(ctx sdk.Context, data json.RawMessage) []abci.ValidatorUpdate {
	var genesisState GenesisState
	ModuleCdc.MustUnmarshalJSON(data, &genesisState)
	InitGenesis(ctx, am.keeper, genesisState)
	return []abci.ValidatorUpdate{}
}

// ExportGenesis returns the exported genesis state as raw bytes for the scavenge
// module.
func (am AppModule) ExportGenesis(ctx sdk.Context) json.RawMessage {
	gs := ExportGenesis(ctx, am.keeper)
	return ModuleCdc.MustMarshalJSON(gs)
}

// BeginBlock returns the begin blocker for the scavenge module.
func (am AppModule) BeginBlock(ctx sdk.Context, req abci.RequestBeginBlock) {
	BeginBlocker(ctx, req, am.keeper)
}

// EndBlock returns the end blocker for the scavenge module. It returns no validator
// updates.
func (AppModule) EndBlock(sdk.Context, abci.RequestEndBlock) []abci.ValidatorUpdate {
	return nil
}
```

Congratulations you have completed the `scavenge` module!

This module is now able to be incorporated into any Cosmos SDK application.

Since we don't want to _just_ build a module but want to build an application that also uses that module, let's go through the process of configuring an app.

# App 

Our `scaffold` utility has already created a pretty complete `app.go` file for us inside of `./app/app.go`. This version of the `app.go` file is meant to be as simple of an app as possible. It contains only the necessary modules needed for using an app that knows about coins \(`bank`\), user accounts \(`auth`\) and securing the application with proof-of-stake \(`staking`\).

One module which is missing but is part of the Cosmos SDK core set of features is `gov`, which allows for a simple form of governance to take place. This process includes making text proposals which can be voted on with coins or with delegation via [liquid democracy](https://en.wikipedia.org/wiki/Liquid_democracy). This module can also be used to update the logic of the application itself, via parameter changes which can affect specific parts of different modules.

Mostly we can just follow the `TODO`s that are marked in the file and add the necessary information about our new module. Afterwards it should look like:

```go
package app

import (
	"encoding/json"
	"io"
	"os"

	abci "github.com/tendermint/tendermint/abci/types"
	"github.com/tendermint/tendermint/libs/log"
	tmos "github.com/tendermint/tendermint/libs/os"
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
	"github.com/cosmos/sdk-tutorials/scavenge/x/scavenge"
)

const appName = "app"

var (
	// default home directories for the application CLI
	DefaultCLIHome = os.ExpandEnv("$HOME/.scavengeCLI")

	// DefaultNodeHome sets the folder where the applcation data and configuration will be stored
	DefaultNodeHome = os.ExpandEnv("$HOME/.scavengeD")

	// NewBasicManager is in charge of setting up basic module elemnets
	ModuleBasics = module.NewBasicManager(
		genutil.AppModuleBasic{},
		auth.AppModuleBasic{},
		bank.AppModuleBasic{},
		staking.AppModuleBasic{},
		distr.AppModuleBasic{},
		params.AppModuleBasic{},
		slashing.AppModuleBasic{},
		supply.AppModuleBasic{},

		scavenge.AppModuleBasic{},
	)
	// account permissions
	maccPerms = map[string][]string{
		auth.FeeCollectorName:     nil,
		distr.ModuleName:          nil,
		staking.BondedPoolName:    {supply.Burner, supply.Staking},
		staking.NotBondedPoolName: {supply.Burner, supply.Staking},
	}
)

// MakeCodec generates the necessary codecs for Amino
func MakeCodec() *codec.Codec {
	var cdc = codec.New()

	ModuleBasics.RegisterCodec(cdc)
	vesting.RegisterCodec(cdc)
	sdk.RegisterCodec(cdc)
	codec.RegisterCrypto(cdc)

	return cdc.Seal()
}

type NewApp struct {
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
	scavengeKeeper scavenge.Keeper

	// Module Manager
	mm *module.Manager

	// simulation manager
	sm *module.SimulationManager
}

// verify app interface at compile time
var _ simapp.App = (*NewApp)(nil)

// NewInitApp is a constructor function for scavengeApp
func NewInitApp(logger log.Logger, db dbm.DB, traceStore io.Writer, loadLatest bool,
	invCheckPeriod uint, baseAppOptions ...func(*bam.BaseApp)) *NewApp {

	// First define the top level codec that will be shared by the different modules
	cdc := MakeCodec()

	// BaseApp handles interactions with Tendermint through the ABCI protocol
	bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

	bApp.SetAppVersion(version.Version)

	keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
		supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, scavenge.StoreKey)

	tkeys := sdk.NewTransientStoreKeys(staking.TStoreKey, params.TStoreKey)

	// Here you initialize your application with the store keys it requires
	var app = &NewApp{
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

	app.scavengeKeeper = scavenge.NewKeeper(
		app.bankKeeper,
		app.cdc,
		keys[scavenge.StoreKey],
	)

	app.mm = module.NewManager(
		genutil.NewAppModule(app.accountKeeper, app.stakingKeeper, app.BaseApp.DeliverTx),
		auth.NewAppModule(app.accountKeeper),
		bank.NewAppModule(app.bankKeeper, app.accountKeeper),
		scavenge.NewAppModule(app.scavengeKeeper, app.bankKeeper),
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
		scavenge.ModuleName,
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

// GenesisState represents chain state at the start of the chain. Any initial state (account balances) are stored here.
type GenesisState map[string]json.RawMessage

func NewDefaultGenesisState() GenesisState {
	return ModuleBasics.DefaultGenesis()
}

func (app *NewApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
	var genesisState GenesisState

	err := app.cdc.UnmarshalJSON(req.AppStateBytes, &genesisState)
	if err != nil {
		panic(err)
	}

	return app.mm.InitGenesis(ctx, genesisState)
}

func (app *NewApp) BeginBlocker(ctx sdk.Context, req abci.RequestBeginBlock) abci.ResponseBeginBlock {
	return app.mm.BeginBlock(ctx, req)
}

func (app *NewApp) EndBlocker(ctx sdk.Context, req abci.RequestEndBlock) abci.ResponseEndBlock {
	return app.mm.EndBlock(ctx, req)
}

func (app *NewApp) LoadHeight(height int64) error {
	return app.LoadVersion(height, app.keys[bam.MainStoreKey])
}

// Codec returns simapp's codec
func (app *NewApp) Codec() *codec.Codec {
	return app.cdc
}

// SimulationManager implements the SimulationApp interface
func (app *NewApp) SimulationManager() *module.SimulationManager {
	return app.sm
}

// ModuleAccountAddrs returns all the app's module account addresses.
func (app *NewApp) ModuleAccountAddrs() map[string]bool {
	modAccAddrs := make(map[string]bool)
	for acc := range maccPerms {
		modAccAddrs[supply.NewModuleAddress(acc).String()] = true
	}

	return modAccAddrs
}
```

Something you might notice near the beginning of the file is that I have renamed the `DefaultCLIHome` and the `DefaultNodeHome`. These are the directories located on your machine where the history of your application and the configuration of your CLI are stored, as well as the encrypted information for the keys you generate on your machine. I renamed them to `scavengeCLI` and `scavengeD` to better reflect our application.

Since we don't want to use the generic commands for our CLI and our application given to us by the `scaffold` command, let's rename the files within `cmd` as well. We will rename `./cmd/appcli/` to `./cmd/scavengeCLI` as well as `./cmd/appd` to `./cmd/scavengeD`. Inside our `./cmd/scavengeCLI/main.go` file we will also update to the following format:

```go
package main

import (
	"fmt"
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
	"github.com/cosmos/cosmos-sdk/x/bank"
	bankcmd "github.com/cosmos/cosmos-sdk/x/bank/client/cli"
	"github.com/cosmos/sdk-tutorials/scavenge/app"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
	amino "github.com/tendermint/go-amino"
	"github.com/tendermint/tendermint/libs/cli"
)

func main() {
	// Configure cobra to sort commands
	cobra.EnableCommandSorting = false

	// Instantiate the codec for the command line application
	cdc := app.MakeCodec()

	// Read in the configuration file for the sdk
	config := sdk.GetConfig()
	config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
	config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
	config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
	config.Seal()

	rootCmd := &cobra.Command{
		Use:   "scavengeCLI",
		Short: "Scavenge Client",
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

	// Add flags and prefix all env exposed with AA
	executor := cli.PrepareMainCmd(rootCmd, "AA", app.DefaultCLIHome)

	err := executor.Execute()
	if err != nil {
		fmt.Printf("Failed executing CLI command: %s, exiting...\n", err)
		os.Exit(1)
	}
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

func registerRoutes(rs *lcd.RestServer) {
	client.RegisterRoutes(rs.CliCtx, rs.Mux)
	app.ModuleBasics.RegisterRESTRoutes(rs.CliCtx, rs.Mux)
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

And within our `./cmd/scavengeD/main.go` we will update to the following format:

```go
package main

import (
	"encoding/json"
	"io"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"

	abci "github.com/tendermint/tendermint/abci/types"
	"github.com/tendermint/tendermint/libs/cli"
	"github.com/tendermint/tendermint/libs/log"
	tmtypes "github.com/tendermint/tendermint/types"
	dbm "github.com/tendermint/tm-db"

	"github.com/cosmos/cosmos-sdk/baseapp"
	"github.com/cosmos/cosmos-sdk/client/debug"
	"github.com/cosmos/cosmos-sdk/client/flags"
	"github.com/cosmos/cosmos-sdk/server"
	"github.com/cosmos/cosmos-sdk/store"
	storetypes "github.com/cosmos/cosmos-sdk/store/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/x/auth"
	genutilcli "github.com/cosmos/cosmos-sdk/x/genutil/client/cli"
	"github.com/cosmos/cosmos-sdk/x/staking"

	app "github.com/cosmos/sdk-tutorials/scavenge/app"
)

const flagInvCheckPeriod = "inv-check-period"

var invCheckPeriod uint

func main() {
	cdc := app.MakeCodec()

	config := sdk.GetConfig()
	config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
	config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
	config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
	config.Seal()

	ctx := server.NewDefaultContext()
	cobra.EnableCommandSorting = false
	rootCmd := &cobra.Command{
		Use:               "scavengeD",
		Short:             "Scavenge Daemon (server)",
		PersistentPreRunE: server.PersistentPreRunEFn(ctx),
	}

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
	rootCmd.AddCommand(AddGenesisAccountCmd(ctx, cdc, app.DefaultNodeHome, app.DefaultCLIHome))
	rootCmd.AddCommand(flags.NewCompletionCmd(rootCmd, true))
	rootCmd.AddCommand(debug.Cmd(cdc))

	server.AddCommands(ctx, cdc, rootCmd, newApp, exportAppStateAndTMValidators)

	// prepare and add flags
	executor := cli.PrepareBaseCmd(rootCmd, "AU", app.DefaultNodeHome)
	rootCmd.PersistentFlags().UintVar(&invCheckPeriod, flagInvCheckPeriod,
		0, "Assert registered invariants every N blocks")
	err := executor.Execute()
	if err != nil {
		panic(err)
	}
}

func newApp(logger log.Logger, db dbm.DB, traceStore io.Writer) abci.Application {
	var cache sdk.MultiStorePersistentCache

	if viper.GetBool(server.FlagInterBlockCache) {
		cache = store.NewCommitKVStoreCacheManager()
	}

	return app.NewInitApp(
		logger, db, traceStore, true, invCheckPeriod,
		baseapp.SetPruning(storetypes.NewPruningOptionsFromString(viper.GetString("pruning"))),
		baseapp.SetMinGasPrices(viper.GetString(server.FlagMinGasPrices)),
		baseapp.SetHaltHeight(viper.GetUint64(server.FlagHaltHeight)),
		baseapp.SetHaltTime(viper.GetUint64(server.FlagHaltTime)),
		baseapp.SetInterBlockCache(cache),
	)
}

func exportAppStateAndTMValidators(
	logger log.Logger, db dbm.DB, traceStore io.Writer, height int64, forZeroHeight bool, jailWhiteList []string,
) (json.RawMessage, []tmtypes.GenesisValidator, error) {

	if height != -1 {
		aApp := app.NewInitApp(logger, db, traceStore, false, uint(1))
		err := aApp.LoadHeight(height)
		if err != nil {
			return nil, nil, err
		}
		return aApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
	}

	aApp := app.NewInitApp(logger, db, traceStore, true, uint(1))

	return aApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
}
```

Finally we need to update our new `cmd` names within our `Makefile`. It should be updated to look like:

```makefile
PACKAGES=$(shell go list ./... | grep -v '/simulation')

VERSION := $(shell echo $(shell git describe --tags) | sed 's/^v//')
COMMIT := $(shell git log -1 --format='%H')

ldflags = -X github.com/cosmos/cosmos-sdk/version.Name=NewApp \
	-X github.com/cosmos/cosmos-sdk/version.ServerName=scavengeD \
	-X github.com/cosmos/cosmos-sdk/version.ClientName=scavengeCLI \
	-X github.com/cosmos/cosmos-sdk/version.Version=$(VERSION) \
	-X github.com/cosmos/cosmos-sdk/version.Commit=$(COMMIT) 

BUILD_FLAGS := -ldflags '$(ldflags)'

.PHONY: all
all: install

.PHONY: install
install: go.sum
		go install -mod=readonly $(BUILD_FLAGS) ./cmd/scavengeD
		go install -mod=readonly $(BUILD_FLAGS) ./cmd/scavengeCLI

go.sum: go.mod
		@echo "--> Ensure dependencies have not been modified"
		GO111MODULE=on go mod verify

# Uncomment when you have some tests
# test:
# 	@go test -mod=readonly $(PACKAGES)
.PHONY: lint
# look into .golangci.yml for enabling / disabling linters
lint:
	@echo "--> Running linter"
	@golangci-lint run
	@go mod verify
```

Now our app is configured and ready to go!

Let's fire it up 🔥

# Run the app

Now that our module is built and our app is configured to use it we can start running our application! The first thing to do is make sure that the `go.mod` is correct. If you're using an IDE like vscode with `golang` extensions enabled, this should be done automatically for you after saving each file. You can also make sure all dependencies are present by running `go mod tidy`.

Once your dependencies are set, run `make` to build your binaries! You will be creating two binaries. One is the `scavengeD` which is a daemon that runs your actual application. The other binary is `scavengeCLI` which is a tool for interacting with your running application.

After you run `make` you want to make sure you have access to both of those binaries. You can do this by running the `scavengeD --help`, where you should see the following:

```text
Scavenge Daemon (server)

Usage:
  scavengeD [command]

Available Commands:
  init                Initialize private validator, p2p, genesis, and application configuration files
  collect-gentxs      Collect genesis txs and output a genesis.json file
  gentx               Generate a genesis tx carrying a self delegation
  validate-genesis    validates the genesis file at the default location or at the location passed as an arg
  add-genesis-account Add genesis account to genesis.json
  start               Run the full node
  unsafe-reset-all    Resets the blockchain database, removes address book files, and resets priv_validator.json to the genesis state
                      
  tendermint          Tendermint subcommands
  export              Export state to JSON
                      
  version             Print the app version
  help                Help about any command

Flags:
  -h, --help                    help for scavengeD
      --home string             directory for config and data (default "/home/billy/.scavengeD")
      --inv-check-period uint   Assert registered invariants every N blocks
      --log_level string        Log level (default "main:info,state:info,*:error")
      --trace                   print out full stack trace on errors

Use "scavengeD [command] --help" for more information about a command.
```

You should also be able to run `scavengeCLI --help` which should result in the following screen:

```text
Scavenge Client

Usage:
  scavengeCLI [command]

Available Commands:
  status      Query remote node for status
  config      Create or query an application CLI configuration file
  query       Querying subcommands
  tx          Transactions subcommands
              
  rest-server Start LCD (light-client daemon), a local REST server
              
  keys        Add or view local private keys
              
  version     Print the app version
  help        Help about any command

Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
  -h, --help              help for scavengeCLI
      --home string       directory for config and data (default "/home/billy/.scavengeCLI")
  -o, --output string     Output format (text|json) (default "text")
      --trace             print out full stack trace on errors

Use "scavengeCLI [command] --help" for more information about a command.
```

Now we should create some users within our app that have some initial coins that can be used as bounties for other players. First we create two users with the following commands:

```text
scavengeCLI keys add me
scavengeCLI keys add you
```

Each command will come with a prompt to set a password to secure the account. I usually use `1234567890` when I'm developing so that I don't forget.

Next you need to initialize your application using the Daemon command with a `<moniker>` \(which is just a nickname for your machine\) and a `<chain-id>` which will be a way to identify your application.

```text
scavengeD init mynode --chain-id scavenge
```

Now you can add your two accounts to the initial state of the application, called the Genesis, using the following commands:

```text
scavengeD add-genesis-account $(scavengeCLI keys show me -a) 1000foo,100000000stake
scavengeD add-genesis-account $(scavengeCLI keys show you -a) 1foo
```

Notice we've combined two commands, which includes one from the Daemon and one from the CLI. The CLI command queries the accounts that we created but displays just their addresses. **Addresses are a bit like user IDs**. You'll also notice that we added some coins to the different users. For the user `me` we added some token called `foo` as well as some token called `stake`. We will be using `stake` within the Proof-Of-Stake validation process. Since the user `me` is the only user with `stake`, they will be the only **Validator** interacting with this application. That's great for our purposes since we're just playing around but if you were to run an application in production you might want to have a more Validators helping to make sure your app runs correctly.

Before we start the application it's good to configure your CLI to know that it will be interacting with this app, and not any other one. These commands will tell the CLI to talk to just this application:

```text
scavengeCLI config chain-id scavenge
scavengeCLI config output json
scavengeCLI config indent true
scavengeCLI config trust-node true
```

Now we want to let the application know that it will be the user `me` who will be validating so we run this command:

```text
scavengeD gentx --name me
```

This command will ask for the password, which if you're like me is just `1234567890`.

Our finaly step is to tell the application that we're done configuring it. This will collect all of our initial configuraiton and prepare the application to begin:

```text
scavengeD collect-gentxs
```

I usually combine all of these commands into a single executable file so that if I make changes to the application I don't have to run each one manually. You can see here that I set the config to use `keyring-backend` to `test` so that we don't need to use a password every time. I put everything into a file called `./init.sh` so that it looks like so:

```bash
#!/bin/bash
rm -r ~/.scavengeCLI
rm -r ~/.scavengeD

scavengeD init mynode --chain-id scavenge

scavengeCLI config keyring-backend test

scavengeCLI keys add me
scavengeCLI keys add you

scavengeD add-genesis-account $(scavengeCLI keys show me -a) 1000foo,100000000stake
scavengeD add-genesis-account $(scavengeCLI keys show you -a) 1foo

scavengeCLI config chain-id scavenge
scavengeCLI config output json
scavengeCLI config indent true
scavengeCLI config trust-node true

scavengeD gentx --name me --keyring-backend test
scavengeD collect-gentxs
```

**Now,** _**finally**_**, you can run your APPLICATION!**

To do so open a new terminal window and type the following:

```text
scavengeD start
```

That's it! You're up and running!

To play with your application take a look at the example commands used to create and solve scavenges.

## Play 

Your application is running! That's great but who cares unless you can play with it. The first command you will want to try is creating a new scavenge. Since our user `me` has way more `foo` token than the user `you`, let's create the scavenge from their account.

You can begin by running `scavengeCLI tx scavenge --help` to see all the commands we created for your new module. You should see the following options:

```javascript
scavenge transactions subcommands

Usage:
  scavengeCLI tx scavenge [flags]
  scavengeCLI tx scavenge [command]

Available Commands:
  createScavenge Creates a new scavenge with a reward
  commitSolution Commits a solution for scavenge
  revealSolution Reveals a solution for scavenge

Flags:
  -h, --help   help for scavenge

Global Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
      --home string       directory for config and data (default "/home/billy/.scavengeCLI")
  -o, --output string     Output format (text|json) (default "text")
      --trace             print out full stack trace on errors

Use "scavengeCLI tx scavenge [command] --help" for more information about a command.
```

We want to use the `createScavenge` command so let's check the help screen for it as well like `scavengeCLI scavenge createScavenge --help`. It should look like:

```text
Creates a new scavenge with a reward

Usage:
  scavengeCLI tx scavenge createScavenge [reward] [solution] [description] [flags]

Flags:
  -a, --account-number uint     The account number of the signing account (offline mode only)
  -b, --broadcast-mode string   Transaction broadcasting mode (sync|async|block) (default "sync")
      --dry-run                 ignore the --gas flag and perform a simulation of a transaction, but don't broadcast it
      --fees string             Fees to pay along with transaction; eg: 10uatom
      --from string             Name or address of private key with which to sign
      --gas string              gas limit to set per-transaction; set to "auto" to calculate required gas automatically (default 200000) (default "200000")
      --gas-adjustment float    adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string       Gas prices to determine the transaction fee (e.g. 10uatom)
      --generate-only           Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase is not accessible and the node operates offline)
  -h, --help                    help for createScavenge
      --indent                  Add indent to JSON response
      --ledger                  Use a connected Ledger device
      --memo string             Memo to send along with transaction
      --node string             <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
  -s, --sequence uint           The sequence number of the signing account (offline mode only)
      --trust-node              Trust connected full node (don't verify proofs for responses) (default true)
  -y, --yes                     Skip tx broadcasting prompt confirmation

Global Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
      --home string       directory for config and data (default "/home/billy/.scavengeCLI")
  -o, --output string     Output format (text|json) (default "text")
```

Let's follow the instructions and create a new scavenge. The first parameter we need is the `reward`. Let's give away `69foo` as a reward for solving our scavenge \(nice\).

Next we should list our `solution`, but probably we should also know what the actual quesiton is that our solution solves \(our `description`\). How about our challenge question be something family friendly like: `What's brown and sticky?`. Of course the only solution to this question is: `A stick`.

Now we have all the pieces needed to create our Message. Let's piece them all together, adding the flag `--from` so the CLI knows who is sending it:

```text
scavengeCLI tx scavenge createScavenge 69foo "A stick" "What's brown and sticky?" --from me
```

After confirming the message looks correct and signing with your password \(`1234567890`?\) you should see something like the following:

```text
{
  "height": "0",
  "txhash": "3D088632B1C523EF2754153F5454E8FA464AE69747A4BD8ABC01A3428C31C185",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\",\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"CreateScavenge\"}]}]}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": "",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "CreateScavenge"
            }
          ]
        }
      ]
    }
  ]
}
```

This tells you that the message was accepted into the app. Whether the message failed afterward can not be told from this screen. However, the section under `txhash` is like a receipt for this interaction. To see if it was successfully processed after being successfully included you can run the following command:

```text
scavengeCLI q tx <txhash>
```

But replace the `<txhash>` with your own. You should see something similar to this afterward:

```text
{
  "height": "2622",
  "txhash": "3D088632B1C523EF2754153F5454E8FA464AE69747A4BD8ABC01A3428C31C185",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\",\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"sender\",\"value\":\"cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer\"},{\"key\":\"module\",\"value\":\"scavenge\"},{\"key\":\"action\",\"value\":\"CreateScavenge\"},{\"key\":\"sender\",\"value\":\"cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer\"},{\"key\":\"description\",\"value\":\"What's brown and sticky?\"},{\"key\":\"solutionHash\",\"value\":\"2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906\"},{\"key\":\"reward\",\"value\":\"69foo\"},{\"key\":\"action\",\"value\":\"CreateScavenge\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg\"},{\"key\":\"amount\",\"value\":\"69foo\"}]}]}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": "",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "sender",
              "value": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer"
            },
            {
              "key": "module",
              "value": "scavenge"
            },
            {
              "key": "action",
              "value": "CreateScavenge"
            },
            {
              "key": "sender",
              "value": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer"
            },
            {
              "key": "description",
              "value": "What's brown and sticky?"
            },
            {
              "key": "solutionHash",
              "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
            },
            {
              "key": "reward",
              "value": "69foo"
            },
            {
              "key": "action",
              "value": "CreateScavenge"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg"
            },
            {
              "key": "amount",
              "value": "69foo"
            }
          ]
        }
      ]
    }
  ],
  "gas_wanted": "200000",
  "gas_used": "28218",
  "tx": {
    "type": "cosmos-sdk/StdTx",
    "value": {
      "msg": [
        {
          "type": "scavenge/CreateScavenge",
          "value": {
            "creator": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer",
            "description": "What's brown and sticky?",
            "solutionHash": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906",
            "reward": [
              {
                "denom": "foo",
                "amount": "69"
              }
            ]
          }
        }
      ],
      "fee": {
        "amount": [],
        "gas": "200000"
      },
      "signatures": [
        {
          "pub_key": {
            "type": "tendermint/PubKeySecp256k1",
            "value": "Ag2Ukd9c1kczh/jpSNHAFZRkm2UfnVb+LHTqQ4SPGAVj"
          },
          "signature": "W222xWoImFlcspUhkb4BImM8WcTfmq8D3pQy83Ceo/109LWcBjRlsU+qrzOf2cC0rUz8EtakQJ5pcmU0+ZSUCQ=="
        }
      ],
      "memo": ""
    }
  },
  "timestamp": "2020-01-18T17:37:36Z",
  "events": [
    {
      "type": "message",
      "attributes": [
        {
          "key": "sender",
          "value": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer"
        },
        {
          "key": "module",
          "value": "scavenge"
        },
        {
          "key": "action",
          "value": "CreateScavenge"
        },
        {
          "key": "sender",
          "value": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer"
        },
        {
          "key": "description",
          "value": "What's brown and sticky?"
        },
        {
          "key": "solutionHash",
          "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
        },
        {
          "key": "reward",
          "value": "69foo"
        },
        {
          "key": "action",
          "value": "CreateScavenge"
        }
      ]
    },
    {
      "type": "transfer",
      "attributes": [
        {
          "key": "recipient",
          "value": "cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg"
        },
        {
          "key": "amount",
          "value": "69foo"
        }
      ]
    }
  ]
}
```

Here you can see all the events we defined within our `Handler` that describes exactly what happened when this message was processed. Since our message was formatted correctly and since the user `me` had enough `foo` to pay the bounty, our `Scavenge` was accepted. You can also see what the solution looks like now that it has been hashed:

```json
{
    "key": "solutionHash",
    "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
}
```

Since we know the solution to this question and since we have another user at hand that can submit it, let's begin the process of committing and revealing that solution.

First we should check the CLI command for `commitSolution` by running `scavengeCLI tx scavenge commitSolution --help` in order to see:

```text
Commits a solution for scavenge

Usage:
  scavengeCLI tx scavenge commitSolution [solution] [flags]

Flags:
  -a, --account-number uint     The account number of the signing account (offline mode only)
  -b, --broadcast-mode string   Transaction broadcasting mode (sync|async|block) (default "sync")
      --dry-run                 ignore the --gas flag and perform a simulation of a transaction, but don't broadcast it
      --fees string             Fees to pay along with transaction; eg: 10uatom
      --from string             Name or address of private key with which to sign
      --gas string              gas limit to set per-transaction; set to "auto" to calculate required gas automatically (default 200000) (default "200000")
      --gas-adjustment float    adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string       Gas prices to determine the transaction fee (e.g. 10uatom)
      --generate-only           Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase is not accessible and the node operates offline)
  -h, --help                    help for commitSolution
      --indent                  Add indent to JSON response
      --ledger                  Use a connected Ledger device
      --memo string             Memo to send along with transaction
      --node string             <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
  -s, --sequence uint           The sequence number of the signing account (offline mode only)
      --trust-node              Trust connected full node (don't verify proofs for responses) (default true)
  -y, --yes                     Skip tx broadcasting prompt confirmation

Global Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
      --home string       directory for config and data (default "/home/billy/.scavengeCLI")
  -o, --output string     Output format (text|json) (default "text")
```

Let's follow the instructions and submit the answer as a commit on behalf of `you`:

```text
scavengeCLI tx scavenge commitSolution "A stick" --from you 
```

We don't need to put the `solutionHash` because it can be generated by hashing our actual solution. After confirming the transaction and signing it we should see our `txhash` again. To confirm the `txhash` let's look at it again with `scavengeCLI q tx <txhash>`. This time you should see something like:

```text
{
  "height": "2733",
  "txhash": "2E27A06BA7047FD41DC0DAD5481D99D5E58BC84DA0D7E0F4E1AC789F7A410186",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\",\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"module\",\"value\":\"scavenge\"},{\"key\":\"action\",\"value\":\"CommitSolution\"},{\"key\":\"sender\",\"value\":\"cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79\"},{\"key\":\"solutionHash\",\"value\":\"2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906\"},{\"key\":\"solutionScavengerHash\",\"value\":\"c65363ed8f6af5d5bd5d9fc9f955106fb7f3356cb218f939a5b658d8a46365a8\"},{\"key\":\"action\",\"value\":\"CommitSolution\"}]}]}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": "",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "module",
              "value": "scavenge"
            },
            {
              "key": "action",
              "value": "CommitSolution"
            },
            {
              "key": "sender",
              "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
            },
            {
              "key": "solutionHash",
              "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
            },
            {
              "key": "solutionScavengerHash",
              "value": "c65363ed8f6af5d5bd5d9fc9f955106fb7f3356cb218f939a5b658d8a46365a8"
            },
            {
              "key": "action",
              "value": "CommitSolution"
            }
          ]
        }
      ]
    }
  ],
  "gas_wanted": "200000",
  "gas_used": "17130",
  "tx": {
    "type": "cosmos-sdk/StdTx",
    "value": {
      "msg": [
        {
          "type": "scavenge/CommitSolution",
          "value": {
            "scavenger": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79",
            "solutionhash": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906",
            "solutionScavengerHash": "c65363ed8f6af5d5bd5d9fc9f955106fb7f3356cb218f939a5b658d8a46365a8"
          }
        }
      ],
      "fee": {
        "amount": [],
        "gas": "200000"
      },
      "signatures": [
        {
          "pub_key": {
            "type": "tendermint/PubKeySecp256k1",
            "value": "AxHwDfJwPnyoTrt5o8L7iSCiUzIOsCOPovWicfgAyIZp"
          },
          "signature": "tUmtFxvNISe8SiUMRAYFkKuDJ58tcMQHfsAU0gZ55ZEcybAwvPou3ggTvVTIxicuI1bwjl7mTiLbplxJMQo6kA=="
        }
      ],
      "memo": ""
    }
  },
  "timestamp": "2020-01-18T17:46:54Z",
  "events": [
    {
      "type": "message",
      "attributes": [
        {
          "key": "module",
          "value": "scavenge"
        },
        {
          "key": "action",
          "value": "CommitSolution"
        },
        {
          "key": "sender",
          "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
        },
        {
          "key": "solutionHash",
          "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
        },
        {
          "key": "solutionScavengerHash",
          "value": "c65363ed8f6af5d5bd5d9fc9f955106fb7f3356cb218f939a5b658d8a46365a8"
        },
        {
          "key": "action",
          "value": "CommitSolution"
        }
      ]
    }
  ]
}
```

You'll notice that the `solutionHash` matches the one before. We've also created a new hash for the `solutionScavengerHash` which is the combination of the solution and our account address. We can make sure the commit has been made by querying it directly as well:

```text
scavengeCLI q scavenge commited "A stick" $(scavengeCLI keys show you -a)
```

Hopefully you should see something like:

```json
{
  "scavenger": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79",
  "solutionHash": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906",
  "solutionScavengerHash": "c65363ed8f6af5d5bd5d9fc9f955106fb7f3356cb218f939a5b658d8a46365a8"
}
```

This confirms that your commit was successfully submitted and is awaiting the follow-up reveal. To make that command let's first check the `--help` command using `scavengeCLI tx scavenge revealSolution --help`. This should show the following screen:

```text
Reveals a solution for scavenge

Usage:
  scavengeCLI tx scavenge revealSolution [solution] [flags]

Flags:
  -a, --account-number uint     The account number of the signing account (offline mode only)
  -b, --broadcast-mode string   Transaction broadcasting mode (sync|async|block) (default "sync")
      --dry-run                 ignore the --gas flag and perform a simulation of a transaction, but don't broadcast it
      --fees string             Fees to pay along with transaction; eg: 10uatom
      --from string             Name or address of private key with which to sign
      --gas string              gas limit to set per-transaction; set to "auto" to calculate required gas automatically (default 200000) (default "200000")
      --gas-adjustment float    adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string       Gas prices to determine the transaction fee (e.g. 10uatom)
      --generate-only           Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase is not accessible and the node operates offline)
  -h, --help                    help for revealSolution
      --indent                  Add indent to JSON response
      --ledger                  Use a connected Ledger device
      --memo string             Memo to send along with transaction
      --node string             <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
  -s, --sequence uint           The sequence number of the signing account (offline mode only)
      --trust-node              Trust connected full node (don't verify proofs for responses) (default true)
  -y, --yes                     Skip tx broadcasting prompt confirmation

Global Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
      --home string       directory for config and data (default "/home/billy/.scavengeCLI")
  -o, --output string     Output format (text|json) (default "text")
      --trace             print out full stack trace on errors
```

Since all we need is the solution again let's send and confirm our final message:

```text
scavengeCLI tx scavenge revealSolution "A stick" --from you
```

We can gather the `txhash` and query it again using `scavengeCLI q tx <txhash>` to reveal:

```json
{
  "height": "2810",
  "txhash": "086B122735C728B2556E04D537E53D6C91C3B46CE0ED0BB6C5001006A4BD2B0F",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\",\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"sender\",\"value\":\"cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg\"},{\"key\":\"module\",\"value\":\"scavenge\"},{\"key\":\"action\",\"value\":\"SolveScavenge\"},{\"key\":\"sender\",\"value\":\"cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79\"},{\"key\":\"solutionHash\",\"value\":\"2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906\"},{\"key\":\"description\",\"value\":\"What's brown and sticky?\"},{\"key\":\"solution\",\"value\":\"A stick\"},{\"key\":\"scavenger\",\"value\":\"cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79\"},{\"key\":\"reward\",\"value\":\"69foo\"},{\"key\":\"action\",\"value\":\"RevealSolution\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79\"},{\"key\":\"amount\",\"value\":\"69foo\"}]}]}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": "",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "sender",
              "value": "cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg"
            },
            {
              "key": "module",
              "value": "scavenge"
            },
            {
              "key": "action",
              "value": "SolveScavenge"
            },
            {
              "key": "sender",
              "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
            },
            {
              "key": "solutionHash",
              "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
            },
            {
              "key": "description",
              "value": "What's brown and sticky?"
            },
            {
              "key": "solution",
              "value": "A stick"
            },
            {
              "key": "scavenger",
              "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
            },
            {
              "key": "reward",
              "value": "69foo"
            },
            {
              "key": "action",
              "value": "RevealSolution"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
            },
            {
              "key": "amount",
              "value": "69foo"
            }
          ]
        }
      ]
    }
  ],
  "gas_wanted": "200000",
  "gas_used": "30740",
  "tx": {
    "type": "cosmos-sdk/StdTx",
    "value": {
      "msg": [
        {
          "type": "scavenge/RevealSolution",
          "value": {
            "scavenger": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79",
            "solutionHash": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906",
            "solution": "A stick"
          }
        }
      ],
      "fee": {
        "amount": [],
        "gas": "200000"
      },
      "signatures": [
        {
          "pub_key": {
            "type": "tendermint/PubKeySecp256k1",
            "value": "AxHwDfJwPnyoTrt5o8L7iSCiUzIOsCOPovWicfgAyIZp"
          },
          "signature": "w0wgQzYL2OeNYsnSJ3WwQo9tNv3RC+Qu6Oo+AdsyFDduASZ4p4vioBxGq/iMn4bNOG5mCKS6eUJZmHN0x6gx9g=="
        }
      ],
      "memo": ""
    }
  },
  "timestamp": "2020-01-18T17:53:21Z",
  "events": [
    {
      "type": "message",
      "attributes": [
        {
          "key": "sender",
          "value": "cosmos13aupkh5020l9u6qquf7lvtcxhtr5jjama2kwyg"
        },
        {
          "key": "module",
          "value": "scavenge"
        },
        {
          "key": "action",
          "value": "SolveScavenge"
        },
        {
          "key": "sender",
          "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
        },
        {
          "key": "solutionHash",
          "value": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906"
        },
        {
          "key": "description",
          "value": "What's brown and sticky?"
        },
        {
          "key": "solution",
          "value": "A stick"
        },
        {
          "key": "scavenger",
          "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
        },
        {
          "key": "reward",
          "value": "69foo"
        },
        {
          "key": "action",
          "value": "RevealSolution"
        }
      ]
    },
    {
      "type": "transfer",
      "attributes": [
        {
          "key": "recipient",
          "value": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
        },
        {
          "key": "amount",
          "value": "69foo"
        }
      ]
    }
  ]
}
```

You'll notice that the final event that was submitted was a transfer. This shows the movement of the reward into the account of the user `you`. To confirm `you` now has `69foo` more you can query their account balance as follows:

```text
scavengeCLI q account $(scavengeCLI keys show you -a)
```

This should show a healthy account balance of `70foo` since `you` began with `1foo`:

```json
{
  "type": "cosmos-sdk/Account",
  "value": {
    "address": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79",
    "coins": [
      {
        "denom": "foo",
        "amount": "70"
      }
    ],
    "public_key": {
      "type": "tendermint/PubKeySecp256k1",
      "value": "AxHwDfJwPnyoTrt5o8L7iSCiUzIOsCOPovWicfgAyIZp"
    },
    "account_number": "1",
    "sequence": "5"
  }
}
```

If you'd like to take a look at the completed scavenge you can first query all scavenges with:

```text
scavengeCLI q scavenge list 
```

To see the specific one just use:

```text
scavengeCLI q scavenge get 2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906
```

Which should show you that the scavenge has in fact been completed:

```json
{
  "creator": "cosmos1uajgdapslnsthwscpy474t3k69u0r8z3u0aaer",
  "description": "What's brown and sticky?",
  "solutionHash": "2f9457a6e8fb202f9e10389a143a383106268c460743dd59d723c0f82d9ba906",
  "reward": [
    {
      "denom": "foo",
      "amount": "69"
    }
  ],
  "solution": "A stick",
  "scavenger": "cosmos1m9pxr3nrra2cl07kh8hzdty5x0mejf44997f79"
}
```

# Conclusion

**Thanks for joining me** in building a deterministic state machine and using it as a game. I hope you can see that even such a simple app can be extremely powerful as it contains digital scarcity.

If you'd like to keep going, consider trying to expand on the capabilities of this application by doing one of the following:

* Allow the `Creator` of a `Scavenge` to edit or delete a scavenge.
* Create a query that lists all commits.

If you're interested in learning more about the Cosmos SDK check out the rest of our [docs](https://docs.cosmos.network/) or join our [forum](https://forum.cosmos.network/).

# Next Steps

Topics to look out for in future tutorials are:

* [Communication between applications \(IBC\)](https://cosmos.network/ibc/)
* [Digital Collectibles \(NFTs\)](https://github.com/cosmos/modules)
* [Using the Ethereum Virtual Machine \(EVM\) as a module within an application](https://github.com/chainsafe/ethermint)

If you have any questions or comments feel free to open an issue on this tutorial's [github](https://github.com/cosmos/sdk-tutorials).

# About the Author

If you'd like to stay in touch with me follow my github at [@okwme](https://github.com/okwme) or twitter at [@billyrennekamp](https://twitter.com/billyrennekamp).

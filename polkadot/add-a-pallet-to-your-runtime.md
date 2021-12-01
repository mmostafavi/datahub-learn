[The original tutorial can be found in Substrate's official documentation here. ](https://substrate.dev/docs/en/tutorials/add-a-pallet/)

# Introduction

The [Substrate Developer Hub Node Template](https://github.com/substrate-developer-hub/substrate-node-template) provides a minimal working runtime which you can use to quickly get started building your own custom blockchain. The Node Template includes [a number of components](https://substrate.dev/docs/en/index#architecture), including a [runtime](https://substrate.dev/docs/en/knowledgebase/getting-started/glossary#runtime) that is constructed using the [FRAME](https://substrate.dev/docs/en/knowledgebase/runtime/frame) runtime development framework. However, in order to remain minimal, it does not include most of the modules \(called "pallets"\) from Substrate's set of core FRAME pallets.

This guide will show you how you can add the [Nicks pallet](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/index.html). You can follow similar patterns to add additional FRAME pallets to your runtime, however you should note that each pallet is a little different in terms of the specific configuration settings needed to use it correctly. This tutorial will help you understand what you'll need to consider when adding a new pallet to your FRAME runtime.

If you run into an issue on this tutorial, **we are here to help!** You can [ask a question on Stack Overflow](https://stackoverflow.com/questions/tagged/substrate) and use the `substrate` tag or contact us on [Element](https://matrix.to/#/#substrate-technical:matrix.org).

### Install the Node Template

You should already have version `v3.0.0` of the Node Template compiled on your computer from when you completed the [Create Your First Substrate Chain](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/) tutorial. If you do not, please complete it.

> Experienced developers who truly prefer to skip those tutorials may install the Node Template according to [the instructions in its readme](https://github.com/substrate-developer-hub/substrate-node-template#getting-started).

## Import the Nicks Pallet <a id="__docusaurus"></a>

We will now modify the Substrate Developer Hub Node Template to include the Nicks pallet; this pallet allows blockchain users to pay a deposit to reserve a nickname and associate it with an account they control.

Open the Node Template in your favorite code editor. We will be editing two files: `runtime/src/lib.rs`, and `runtime/Cargo.toml`.

```rust
substrate-node-template
|
+-- runtime
|   |
|   +-- Cargo.toml    <-- One change in this file
|   |
|   +-- build.rs
|   |
|   +-- src
|       |
|       +-- lib.rs     <-- Most changes in this file
|
+-- pallets
|
+-- scripts
|
+-- node
|
+-- ...
Copy
```

### Importing a Pallet Crate

The first thing you need to do to add the Nicks pallet is to import the `pallet-nicks` crate in your runtime's `Cargo.toml` file. If you want a proper primer into Cargo References, you should check out [their official documentation](https://doc.rust-lang.org/cargo/reference/index.html).

Open `substrate-node-template/runtime/Cargo.toml` and you will see a list of all the dependencies your runtime has. For example, it depends on the [Balances pallet](https://substrate.dev/rustdocs/v3.0.0/pallet_balances/index.html):

**`runtime/Cargo.toml`**

```text
[dependencies]
#--snip--
pallet-balances = { default-features = false, version = '3.0.0' }
Copy
```

#### Crate Features

One important thing we need to call out with importing pallet crates is making sure to set up the crate `features` correctly. In the code snippet above, you will notice that we set `default_features = false`. If you explore the `Cargo.toml` file even closer, you will find something like:

**`runtime/Cargo.toml`**

```rust
[features]
default = ['std']
std = [
    'codec/std',
    'serde',
    'frame-executive/std',
    'frame-support/std',
    'frame-system/std',
    'frame-system-rpc-runtime-api/std',
    'pallet-aura/std',
    'pallet-balances/std',
    #--snip--
]
Copy
```

This second line defines the `default` features of your runtime crate as `std`. You can imagine, each pallet crate has a similar configuration defining the default feature for the crate. Your feature will determine the features that should be used on downstream dependencies. For example, the snippet above should be read as:

> The default feature for this Substrate runtime is `std`. When `std` feature is enabled for the runtime, `codec`, `serde`, `frame-executive`, `frame-support`, and all the other listed dependencies should use their `std` feature too.

This is important to enable the Substrate runtime to compile to both native binaries, which support Rust [`std`](https://doc.rust-lang.org/std/), and [Wasm](https://webassembly.org/) binaries, which do not \(see: [`no_std`](https://rust-embedded.github.io/book/intro/no-std.html)\).

The use of Wasm runtime binaries is one of Substrate's defining features. It allows the runtime code to become a part of a blockchain's evolving state; it also means that the definition of the runtime itself is subject to the cryptographic consensus mechanisms that ensure the security of the blockchain network. The usage of Wasm runtimes enables one of Substrate's most innovative features: forkless runtime upgrades, which means that Substrate blockchain nodes can stay up-to-date and even acquire new features without needing to be replaced with an updated application binary.

To see how the `std` and `no_std` features are actually used in the runtime code, we can open the project file:

**`runtime/src/lib.rs`**

```rust
#![cfg_attr(not(feature = "std"), no_std)]
// `construct_runtime!` does a lot of recursion and requires us to increase the limit to 256.
#![recursion_limit="256"]

// Make the WASM binary available.
#[cfg(feature = "std")]
include!(concat!(env!("OUT_DIR"), "/wasm_binary.rs"));

// --snip--
Copy
```

You can see that at the top of the file, we define that we will use `no_std` when we are _not_ using the `std` feature. A few lines lower you can see `#[cfg(feature = "std")]` above the `wasm_binary.rs` import, which is a flag saying to only import the Wasm binary when we have enabled the `std` feature.

#### Importing the Nicks Pallet Crate

Okay, now that we have covered the basics of crate features, we can actually import the Nicks pallet. The Nicks pallet is one of the simpler pallets in FRAME, so it makes for a good example of the common points you need to consider when adding a pallet to your runtime.

First we will add the new dependency by simply copying an existing pallet and changing the values. So based on the `balances` import shown above, the `nicks` import will look like:

**`runtime/Cargo.toml`**

```rust
[dependencies]
#--snip--
pallet-nicks = { default-features = false, version = '3.0.0' }
Copy
```

As with other pallets, the Nicks pallet has an `std` feature. We should build its `std` feature when the runtime is built with its own `std` feature. Add the following line to the runtime's `std` feature.

**`runtime/Cargo.toml`**

```rust
[features]
default = ["std"]
std = [
    #--snip--
    'pallet-nicks/std',
    #--snip--
]
Copy
```

If you forget to set the feature, when building to your native binaries you will get errors like:

```javascript
error[E0425]: cannot find function `memory_teardown` in module `sandbox`
  --> ~/.cargo/git/checkouts/substrate-7e08433d4c370a21/83a6f1a/primitives/sandbox/src/../without_std.rs:53:12
   |
53 |         sandbox::memory_teardown(self.memory_idx);
   |                  ^^^^^^^^^^^^^^^ not found in `sandbox`

error[E0425]: cannot find function `memory_new` in module `sandbox`
  --> ~/.cargo/git/checkouts/substrate-7e08433d4c370a21/83a6f1a/primitives/sandbox/src/../without_std.rs:72:18
   |
72 |         match sandbox::memory_new(initial, maximum) {
   |

...
Copy
```

Before moving on, check that the new dependencies resolve correctly by running:

```text
cargo check -p node-template-runtime
```

## Configure the Nicks Pallet <a id="__docusaurus"></a>

Every pallet has a component called `Config` that is used for configuration. This component is a [Rust "trait"](https://doc.rust-lang.org/book/ch10-02-traits.html); traits in Rust are similar to interfaces in languages such as C++, Java and Go. FRAME developers must implement this trait for each pallet they would like to include in a runtime in order to configure that pallet with the parameters and types that it needs from the outer runtime. For instance, in the template pallet that is included in the [Node Template](https://github.com/substrate-developer-hub/substrate-node-template), you will see the following `Config` configuration trait:

**`pallets/template/src/lib.rs`**

```rust
/// Configure the pallet by specifying the parameters and types on which it depends.
pub trait Config: frame_system::Config {
    /// Because this pallet emits events, it depends on the runtime's definition of an event.
    type Event: From<Event<Self>> + Into<<Self as frame_system::Config>::Event>;
}
Copy
```

As the comment states, we are using the `Event` type of the `Config` configuration trait in order to allow the `TemplateModule` pallet to emit a type of event that is compatible with the outer runtime. The `Config` configuration trait may also be used to tune parameters that control the resources required to interact with a pallet or even to limit the resources of the runtime that the pallet may consume. You will see examples of both such cases below when you implement the `Config` configuration trait for the Nicks pallet.

To figure out what you need to implement for the Nicks pallet specifically, you can take a look at the [`pallet_nicks::Config` documentation](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/trait.Config.html) or the definition of the trait itself in [the source code](https://github.com/paritytech/substrate/blob/v3.0.0/frame/nicks/src/lib.rs) of the Nicks pallet. We have annotated the source code for `nicks` pallet _from the substrate source_ below with enhanced comments that expand on those already included in the documentation, so be sure to read this:

**`substrate/frame/nicks/src/lib.rs`**

```javascript
/// Already in the Nicks pallet included substrate (with enhanced comments):
pub trait Config: frame_system::Config {
    // The runtime must supply this pallet with an Event type that satisfies the pallet's requirements.
    type Event: From<Event<Self>> + Into<<Self as frame_system::Config>::Event>;

    // The currency type that will be used to place deposits on nicks.
    // It must implement ReservableCurrency.
    // https://substrate.dev/rustdocs/v3.0.0/frame_support/traits/trait.ReservableCurrency.html
    type Currency: ReservableCurrency<Self::AccountId>;

    // The amount required to reserve a nick.
    type ReservationFee: Get<BalanceOf<Self>>;

    // A callback that will be invoked when a deposit is forfeited.
    type Slashed: OnUnbalanced<NegativeImbalanceOf<Self>>;

    // Origins are used to identify network participants and control access.
    // This is used to identify the pallet's admin.
    // https://substrate.dev/docs/en/knowledgebase/runtime/origin
    type ForceOrigin: EnsureOrigin<Self::Origin>;

    // This parameter is used to configure a nick's minimum length.
    type MinLength: Get<usize>;

    // This parameter is used to configure a nick's maximum length.
    // https://substrate.dev/docs/en/knowledgebase/runtime/storage#create-bounds
    type MaxLength: Get<usize>;
}
Copy
```

Just like we used the Balances pallet as a template for importing the Nicks pallet, let's use the Balances pallet as an example to help us understand how we can implement the `Config` interface for the Nicks pallet. You will notice that this implementation consists of two parts: a `parameter_types!` block where constant values are defined and an `impl` block where the types and values defined by the `Config` interface are configured. This code block has also been annotated with additional comments that you should be sure to read:

**`runtime/src/lib.rs`**

```javascript
/// Already in your template for Ballances:
parameter_types! {
    // The u128 constant value 500 is aliased to a type named ExistentialDeposit.
    pub const ExistentialDeposit: u128 = 500;
    // A heuristic that is used for weight estimation.
    pub const MaxLocks: u32 = 50;
}

impl pallet_balances::Config for Runtime {
    // The previously defined parameter_type is used as a configuration parameter.
    type MaxLocks = MaxLocks;

    // The "Balance" that appears after the equal sign is an alias for the u128 type.
    type Balance = Balance;

    // The empty value, (), is used to specify a no-op callback function.
    type DustRemoval = ();

    // The previously defined parameter_type is used as a configuration parameter.
    type ExistentialDeposit = ExistentialDeposit;

    // The FRAME runtime system is used to track the accounts that hold balances.
    type AccountStore = System;

    // No weight information is supplied to the Balances pallet by the Node Template's runtime.
    type WeightInfo = ();

    // The ubiquitous event type.
    type Event = Event;
}
Copy
```

The `impl pallet_balances::Config` block allows runtime developers that are including the Balances pallet in their runtime to configure the types and parameters that are specified by the Balances pallet `Config` configuration trait. For example, the `impl` block above configures the Balances pallet to use the `u128` type to track balances. If you were developing a chain where it was important to optimize storage, you could use any unsigned integer type that was at least 32-bits in size; this is because [the `Balance` type](https://substrate.dev/rustdocs/v3.0.0/pallet_balances/pallet/trait.Config.html#associatedtype.Balance) for the Balances pallet `Config` configuration trait is "bounded" by [the `AtLeast32BitUnsigned` trait](https://substrate.dev/rustdocs/v3.0.0/sp_arithmetic/traits/trait.AtLeast32BitUnsigned.html).

Now that you have an idea of the purpose behind the `Config` configuration trait and how you can implement a FRAME pallet's `Config` interface for your runtime, let's implement the `Config` configuration trait for the Nicks pallet. Add the following code to your runtime:

**`runtime/src/lib.rs`**

```javascript
/// Add this code block to your template for Nicks:
parameter_types! {
    // Choose a fee that incentivizes desireable behavior.
    pub const NickReservationFee: u128 = 100;
    pub const MinNickLength: usize = 8;
    // Maximum bounds on storage are important to secure your chain.
    pub const MaxNickLength: usize = 32;
}

impl pallet_nicks::Config for Runtime {
    // The Balances pallet implements the ReservableCurrency trait.
    // https://substrate.dev/rustdocs/v3.0.0/pallet_balances/index.html#implementations-2
    type Currency = pallet_balances::Module<Runtime>;

    // Use the NickReservationFee from the parameter_types block.
    type ReservationFee = NickReservationFee;

    // No action is taken when deposits are forfeited.
    type Slashed = ();

    // Configure the FRAME System Root origin as the Nick pallet admin.
    // https://substrate.dev/rustdocs/v3.0.0/frame_system/enum.RawOrigin.html#variant.Root
    type ForceOrigin = frame_system::EnsureRoot<AccountId>;

    // Use the MinNickLength from the parameter_types block.
    type MinLength = MinNickLength;

    // Use the MaxNickLength from the parameter_types block.
    type MaxLength = MaxNickLength;

    // The ubiquitous event type.
    type Event = Event;
}
Copy
```

#### Adding Nicks to the `construct_runtime!` Macro

Next, we need to add the Nicks pallet to the `construct_runtime!` macro. For this, we need to determine the types that the pallet exposes so that we can tell the runtime that they exist. The complete list of possible types can be found in the [`construct_runtime!` macro documentation](https://substrate.dev/rustdocs/v3.0.0/frame_support/macro.construct_runtime.html).

If we look at the Nicks pallet in detail, we know it has:

* Module **Storage**: Because it uses the `decl_storage!` macro.
* Module **Event**s: Because it uses the `decl_event!` macro. You will notice that in the case of the Nicks pallet, the `Event` keyword is parameterized with respect to a type, `T`; this is because at least one of the events defined by the Nicks pallet depends on a type that is configured with the `Config` configuration trait.
* **Call**able Functions: Because it has dispatchable functions in the `decl_module!` macro.
* The **Module** type from the `decl_module!` macro.

Thus, when we add the pallet, it will look like this:

**`runtime/src/lib.rs`**

```javascript
construct_runtime!(
    pub enum Runtime where
        Block = Block,
        NodeBlock = opaque::Block,
        UncheckedExtrinsic = UncheckedExtrinsic
    {
        /* --snip-- */

        /*** Add This Line ***/
        Nicks: pallet_nicks::{Module, Call, Storage, Event<T>},
    }
);
Copy
```

Note that not all pallets will expose all of these runtime types, and some may expose more! You should always look at the documentation or source code of a pallet to determine which of these types you need to expose.



## Interact with the Nicks Pallet <a id="__docusaurus"></a>

Now you are ready to compile and run your node that has been enhanced with nickname capabilities from the Nicks pallet. Compile the node in release mode with:

```text
cargo build --release
Copy
```

If the build fails, go back to the previous section and make sure you followed all the steps correctly. After the build succeeds, you can start the node:

```text
# Run a temporary node in development mode
./target/release/node-template --dev --tmp
Copy
```

### Start the Front-End

As in the previous tutorials, this tutorial will use the Substrate Developer Hub Front-End Template to allow you to interact with the Node Template. As long as you have completed the [Create Your First Chain](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain) and [Build a dApp](https://substrate.dev/docs/en/tutorials/build-a-dapp) tutorials, you should already be prepared to continue with the rest of this tutorial.

> Refer directly to the [front-end setup instructions](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/setup#install-the-front-end-template) for the Create Your First Chain Tutorial if necessary.

To start the Front-End Template, navigate to its directory and run:

```text
yarn start
Copy
```

### Use the Nicks Pallet

You should already be familiar with using the Front-End Template to [interact with a pallet](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/interact#pallet-interactor-&-events). In this section we will use the Nicks pallet to further illustrate how the Front-End Template can be used to interact with FRAME pallets. We will also learn more about how to use the Front-End Template to invoke privileged functions with the Sudo pallet, which is included by default as part of the Node Template. Finally, you will learn how to interpret the different types of events and errors that FRAME pallets may emit.

To get started, use the account selector from the Front-End Template to select `Alice`'s account and then use the Pallet Interactor component to call [the `setName` dispatchable](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/enum.Call.html#variant.set_name) function from the `nicks` pallet. You can select any name you'd like as long as it is no shorter than the `MinNickLength` and no longer than the `MaxNickLength` you configured in the previous step. Use the `Signed` button to execute the function.

![Set a Name](https://substrate.dev/docs/assets/tutorials/add-a-pallet/set-name.png)

As you can see in the image above, the Front-End Template will report the status of the dispatchable, as well as allow you to observe the [events](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/enum.RawEvent.html) emitted by the Nicks pallet and the other pallets that compose your chain's runtime. Now use the Pallet Interactor's Query capabilities to read the value of Alice's nickname from the [runtime storage](https://substrate.dev/docs/en/knowledgebase/runtime/storage) of the Nicks pallet.

![Read a Name](https://substrate.dev/docs/assets/tutorials/add-a-pallet/name-of-alice.png)

The return type is a tuple that contains two values: Alice's hex-encoded nickname and the amount that was reserved from Alice's account in order to secure the nickname. If you query the Nicks pallet for Bob's nickname, you'll see that the `None` value is returned. This is because Bob has not invoked the `setName` dispatchable and deposited the funds needed to reserve a nickname.

![Read an Empty Name](https://substrate.dev/docs/assets/tutorials/add-a-pallet/name-of-bob.png)

Use the `Signed` button to invoke [the `killName` dispatchable](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/enum.Call.html#variant.kill_name) function and use Bob's account ID as the function's argument. The `killName` function must be called by the `ForceOrigin` that was configured with the Nicks pallet's `Config` interface in the previous section. You may recall that we configured this to be the FRAME system's `Root` origin. The Node Template's [chain specification](https://github.com/substrate-developer-hub/substrate-node-template/blob/v3.0.0/node/src/chain_spec.rs) file is used to configure the [Sudo pallet](https://substrate.dev/rustdocs/v3.0.0/pallet_sudo/index.html) to give Alice access to this origin.

The front-end template makes it easy to use the Sudo pallet to dispatch a call from the `Root` origin - just use the `SUDO` button to invoke the dispatchable. Since we just used the `Signed` button as opposed to the `SUDO` button, the function was _dispatched_ by [the `Signed` origin](https://substrate.dev/rustdocs/v3.0.0/frame_system/enum.RawOrigin.html#variant.Signed) associated with Alice's account as opposed to the `Root` origin.

![\`BadOrigin\` Error](https://substrate.dev/docs/assets/tutorials/add-a-pallet/kill-name-bad-origin.png)

You will notice that even though the function call was successfully dispatched, a `BadOrigin` error was emitted and is visible in the Events pane. This means that Alice's account was still charged [fees](https://substrate.dev/docs/en/knowledgebase/runtime/fees) for the dispatch, but there weren't any state changes executed because the Nicks pallet follows the important [verify-first-write-last](https://substrate.dev/docs/en/knowledgebase/runtime/storage#verify-first-write-last) pattern. Now use the `SUDO` button to dispatch the same call with the same parameter.

![Nicks Pallet Error](https://substrate.dev/docs/assets/tutorials/add-a-pallet/clear-name-error.png)

The Sudo pallet emits a [`Sudid` event](https://substrate.dev/rustdocs/v3.0.0/pallet_sudo/enum.RawEvent.html#variant.Sudid) to inform network participants that the `Root` origin dispatched a call, however, you will notice that the inner dispatch failed with a [`DispatchError`](https://substrate.dev/rustdocs/v3.0.0/sp_runtime/enum.DispatchError.html) \(the Sudo pallet's [`sudo` function](https://substrate.dev/rustdocs/v3.0.0/pallet_sudo/enum.Call.html#variant.sudo) is the "outer" dispatch\). In particular, this was an instance of [the `DispatchError::Module` variant](https://substrate.dev/rustdocs/v3.0.0/frame_support/dispatch/enum.DispatchError.html#variant.Module), which reports two pieces of metadata: an `index` number and an `error` number. The `index` number relates to the pallet from which the error originated; it corresponds with the _index_ \(position\) of the pallet within the `construct_runtime!` macro. The `error` number corresponds with the index of the relevant variant from that pallet's `Error` enum. When using these numbers to find pallet errors, remember that the _first_ position corresponds with index _zero_. In the screenshot above, the `index` is `9` \(the _tenth_ pallet\) and the `error` is `2` \(the _third_ error\). Depending on the position of the Nicks pallet in your `construct_runtime!` macro, you may see a different number for `index`. Regardless of the value of `index`, you should see that the `error` value is `2`, which corresponds to the _third_ variant of the Nick's pallet's `Error` enum, [the `Unnamed` variant](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/enum.Error.html#variant.Unnamed). This shouldn't be a surprise since Bob has not yet reserved a nickname, thus it cannot be cleared!

You should confirm that Alice can use the `SUDO` button to invoke the `killName` dispatchable and forcibly clear the nickname associated with any account \(including her own\) that actually has a nickname associated with it. Here are some other things you may want to try:

* Add a nickname that is shorter than the `MinNickLength` or longer than the `MaxNickLength` that you configured with the Nick's pallet's `Config` configuration trait.
* Add a nickname for Bob then use Alice's account and the `SUDO` button to forcibly kill Bob's nickname. Switch back to Bob's account and dispatch the `clearName` function.

### Adding Other FRAME Pallets

In this guide, we walked through specifically how to import the Nicks pallet, but as mentioned in the beginning of this guide, each pallet will be a little different. Have no fear, you can always refer to the [demonstration Substrate node runtime](https://github.com/paritytech/substrate/tree/v3.0.0/bin/node/runtime) which includes nearly every pallet in the library of core FRAME pallets.

In the `Cargo.toml` file of the Substrate node runtime, you will see an example of how to import each of the different pallets, and in the `lib.rs` file you will see how to add each pallet to your runtime. You can generally copy what was done there as a starting point to include a pallet in your own runtime.

#### Learn More

* Learn how to add a more complex pallet to the Node Template by completing the [Add the Contracts Pallet](https://substrate.dev/docs/en/tutorials/add-contracts-pallet) tutorial.
* Complete the [Forkless Upgrade a Chain](https://substrate.dev/docs/en/tutorials/forkless-upgrade) tutorial to learn how Substrate enables forkless runtime upgrades and follow steps to perform two upgrades, each of which is performed by way of a distinct upgrade mechanism.

## References

* [Nicks pallet docs](https://substrate.dev/rustdocs/v3.0.0/pallet_nicks/index.html)


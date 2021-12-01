[The original tutorial can be found in Substrate's official documentation here. ](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/)

# Introduction

In this tutorial, you will learn how to compile and launch the Substrate Developer Hub Node Template and interact with it using the Substrate Developer Hub Front-End Template.

This tutorial is aimed at someone who has never touched Substrate before and wants to get a **basic** and **quick** understanding of what Substrate is all about. We will not be going into depth about the intricacies of developing on Substrate, but will hopefully satisfy your curiosity so that you will continue your journey!

This tutorial should take you about **1 hour** to complete. We will be compiling [Rust](https://www.rust-lang.org/) code, but you do _not_ need to have any Rust experience to complete this guide. We provide you with working code and explain how to use it.

We only expect that:

* You are generally familiar with software development and command line interfaces.
* You are generally familiar with blockchains and smart contract platforms.
* You are open to learning about the bleeding edge of blockchain development.

If you run into an issue on this tutorial, **we are here to help!** You can [ask a question on Stack Overflow](https://stackoverflow.com/questions/tagged/substrate) and use the `substrate` tag or contact us on [Element](https://matrix.to/#/#substrate-technical:matrix.org).

### What you will be doing

Before we even get started, let's lay out what we are going to do over the course of this tutorial. We will:

1. Set up your computer to be able to develop with Substrate.
2. Use a node template project to run a Substrate-based blockchain.
3. Use a front-end template project to interact with the template node.

Sound reasonable? Good, then let's begin!

## Set Up Your Computer

Normally, we would teach you more about the Substrate blockchain development framework, however, setting up your computer for Substrate development can take a while.

To optimize your time, we will have you start the setup process. While things are compiling, you can continue to the next section to learn more about Substrate and what we are building.

# Prerequisites


You will probably need to do some set-up to prepare your computer for Substrate development.

#### Substrate Development

> Before you continue, complete the [official Installation guide](https://substrate.dev/docs/en/knowledgebase/getting-started/). After `rustup` has been installed and configured, and you've configured the Rust toolchain to default to the latest stable version you can return to these steps.

### Compiling Substrate

Once the prerequisites are installed, you can use Git to clone the Substrate Developer Hub Node Template, which serves as a good starting point for building on Substrate.

1. Clone the Node Template \(version `v3.0.0`\).

   ```text
   git clone -b v3.0.0 --depth 1 https://github.com/substrate-developer-hub/substrate-node-template
   Copy
   ```

2. Compile the Node Template

   ```text
   cd substrate-node-template
   # NOTE: you should always use the `--release` flag
   cargo build --release
   # ^^ this will take a while!
   Copy
   ```

> The time required for the compilation step depends on the hardware you're using.
>
> **You should start building the node template** _**before**_ **moving on!**

### Install the Front-End Template

This tutorial uses a [ReactJS](https://reactjs.org/) front-end template to allow you to interact with the Substrate-based blockchain node that you should have started compiling in the previous step. You can use this same front-end template to create UIs for your own projects in the future.

To use the front-end template, you need [Yarn](https://yarnpkg.com/), which itself requires [Node.js](https://nodejs.org/). If you don't have these tools, you _must_ install them from these instructions:

* [Install Node.js](https://nodejs.org/en/download/)
* [Install Yarn](https://yarnpkg.com/lang/en/docs/install/)

Now you can proceed to set up the front-end template with these commands.

```text
# Clone the frontend template from github
git clone -b v3.0.0 --depth 1 https://github.com/substrate-developer-hub/substrate-front-end-template

# Install the dependencies
cd substrate-front-end-template
yarn install
```

## Background Information 

 While we wait for the [node template to build](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/setup/#compiling-substrate), let's learn a little bit about the Substrate blockchain development framework. The Node Template that you are compiling was built with this framework.

### Substrate

Substrate is an **open source**, **modular**, and **extensible** framework for building blockchains.

Substrate has been designed from the ground up to be flexible and allow innovators to design and build a blockchain network that meets their needs. It provides all the core components you need to build a customized blockchain node.

#### Substrate Developer Hub Node Template

We provide an out-of-the-box working Substrate-based node in the form of the Node Template, which should be compiling as you read this. Without making any changes, you and your friends could share this node template and create a working blockchain network with a cryptocurrency and everything!

We will teach you how to use this node in "development" mode, which allows you to run a network with a single node, and have some pre-configured user accounts with funds.

## Interacting With Your Node 

Now that your node has finished compiling, let's show you how everything works out of the box.

### Starting Your Node

Run the following command to start your node:

```text
# Run a temporary node in development mode
./target/release/node-template --dev --tmp
Copy
```

Note the flags:

* `--dev` this set ups a developer node [chain specification](https://substrate.dev/docs/en/knowledgebase/integrate/chain-spec)
* `--tmp` this saves all active data for the node \(keys, blockchain database, networking info, ...\) and is deleted as soon as you properly terminate your node. So every time you start with this command, you will have a clean state to work from. If the node is killed, `/tmp` is cleaned automatically on the restart of your computer for linux based OSs, and these files can manually be removed if needed.

With this command, you should see something like this if your node is running successfully:

```text
2021-03-16 10:56:51  Running in --dev mode, RPC CORS has been disabled.
2021-03-16 10:56:51  Substrate Node
2021-03-16 10:56:51  ✌️  version 3.0.0-8370ddd-x86_64-linux-gnu
2021-03-16 10:56:51  ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021
2021-03-16 10:56:51  📋 Chain specification: Development
2021-03-16 10:56:51  🏷 Node name: few-size-5380
2021-03-16 10:56:51  👤 Role: AUTHORITY
2021-03-16 10:56:51  💾 Database: RocksDb at /tmp/substrateP1jD7H/chains/dev/db
2021-03-16 10:56:51  ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)
2021-03-16 10:56:51  🔨 Initializing Genesis block/state (state: 0x17df…04a0, header-hash: 0xc43b…ed16)
2021-03-16 10:56:51  👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
2021-03-16 10:56:51  ⏱  Loaded block-time = 6000 milliseconds from genesis on first-launch
2021-03-16 10:56:51  Using default protocol ID "sup" because none is configured in the chain specs
2021-03-16 10:56:51  🏷 Local node identity is: 12D3KooWQdU84EJCqDr4aqfhb7dxXU2fzd6i2Rn1XdNtsiM5jvEC
2021-03-16 10:56:51  📦 Highest known block at #0
2021-03-16 10:56:51  〽️ Prometheus server started at 127.0.0.1:9615
2021-03-16 10:56:51  Listening for new connections on 127.0.0.1:9944.
2021-03-16 10:56:54  🙌 Starting consensus session on top of parent 0xc43b4514877d7dcfff2459cdfe609a96cf8e9b9723589635d7215de6bf00ed16
2021-03-16 10:56:54  🎁 Prepared block for proposing at 1 [hash: 0x255bcf44df92dd4ccaca15d92d4a3db9d276e42843e21ab0cc840e207b2649d6; parent_hash: 0xc43b…ed16; extrinsics (1): [0x02bf…2cbd]]
2021-03-16 10:56:54  🔖 Pre-sealed block for proposal at 1. Hash now 0x9c14d9caccc37f8142fc348d184fb4bd8a8bc217a8979493d7f46d4220775616, previously 0x255bcf44df92dd4ccaca15d92d4a3db9d276e42843e21ab0cc840e207b2649d6.
2021-03-16 10:56:54  ✨ Imported #1 (0x9c14…5616)
2021-03-16 10:56:54  🙌 Starting consensus session on top of parent 0x9c14d9caccc37f8142fc348d184fb4bd8a8bc217a8979493d7f46d4220775616
2021-03-16 10:56:54  🎁 Prepared block for proposing at 2 [hash: 0x6cd4bd9d2a531750c10610bdaa5af0075745b6612ffa3623c14d699250b4e732; parent_hash: 0x9c14…5616; extrinsics (1): [0x3cc8…b8d9]]
2021-03-16 10:56:54  🔖 Pre-sealed block for proposal at 2. Hash now 0x05bd3317b51d717163dfa8847369d7f697c6180868c29f02d0b7ff79c5bbde3f, previously 0x6cd4bd9d2a531750c10610bdaa5af0075745b6612ffa3623c14d699250b4e732.
2021-03-16 10:56:54  ✨ Imported #2 (0x05bd…de3f)
2021-03-16 10:56:56  💤 Idle (0 peers), best: #2 (0x05bd…de3f), finalized #0 (0xc43b…ed16), ⬇ 0 ⬆ 0
2021-03-16 10:57:00  🙌 Starting consensus session on top of parent 0x05bd3317b51d717163dfa8847369d7f697c6180868c29f02d0b7ff79c5bbde3f
2021-03-16 10:57:00  🎁 Prepared block for proposing at 3 [hash: 0xa6990964cf4f184edc08acd61c3c01ac8975abbba6d42f4eec3f9658097aec04; parent_hash: 0x05bd…de3f; extrinsics (1): [0xd6ed…86a5]]
2021-03-16 10:57:00  🔖 Pre-sealed block for proposal at 3. Hash now 0xbe07e322ca525e580a3703637db191c6df091b0242a411b88fa0c43ef0ac31f8, previously 0xa6990964cf4f184edc08acd61c3c01ac8975abbba6d42f4eec3f9658097aec04.
2021-03-16 10:57:00  ✨ Imported #3 (0xbe07…31f8)
2021-03-16 10:57:01  💤 Idle (0 peers), best: #3 (0xbe07…31f8), finalized #1 (0x9c14…5616), ⬇ 0 ⬆ 0
Copy
```

If the number after `finalized:` is increasing, your blockchain is producing new blocks and reaching consensus about the state they describe!

A few notes on the example output above:

* `💾 Database: RocksDb at /tmp/substrateP1jD7H/chains/dev/db` : where chain data is being written
* `🏷 Local node identity is: 12D3KooWQdU84EJCqDr4aqfhb7dxXU2fzd6i2Rn1XdNtsiM5jvEC`: the node ID needed if you intend to connect to other nodes directly \(more in the [private network tutorial](https://substrate.dev/docs/en/tutorials/start-a-private-network/index)\)

> While not critical now, please do read all the startup logs for your node, as they help inform you of key configuration information as you continue to learn and move past these first basic tutorials.

### Start the Front-End Template

To interact with your local node, we will use [the Substrate Developer Hub Front-End Template](https://github.com/substrate-developer-hub/substrate-front-end-template), which is a collection of UI components that have been designed with common use cases in mind.

> Be sure to use the correct version of the template for the version of substrate you are using as [major versions](https://semver.org/) are _not_ expected to be interoperable!

You already installed the Front-End Template; let's launch it by executing the following command in the root directory of the Front-End Template:

```text
# Make sure to run this command in the root directory of the Front-End Template
yarn start
Copy
```

### Interact

Once the Front-End Template is running and loaded in your browser at [http://localhost:8000/](http://localhost:8000/), take a moment to explore its components. At the top, you will find lots of helpful information about the chain to which you're connected as well as an account selector that will let you control the account you use to perform on-chain operations.

![Chain Data &amp; Account Selector](https://substrate.dev/docs/assets/tutorials/first-chain/chain-data.png)

There is also a table that lists [the well-known test accounts](https://substrate.dev/docs/en/knowledgebase/integrate/subkey#well-known-keys) that you have access to. Some, like Alice and Bob, already have funds!

![Account Table](https://substrate.dev/docs/assets/tutorials/first-chain/accts-prefunded.png)

Beneath the account table there is a Transfer component that you can use to transfer funds from one account to another. Take note of the info box that describes the precision used by the Front-End Template; you should transfer at least `1000000000000` to make it easy for you to observe the changes you're making. Here we give the `dave` development account these funds:

![Balance Transfer](https://substrate.dev/docs/assets/tutorials/first-chain/apps-transfer.png)

Notice that the table of accounts is dynamic and that the account balances are updated as soon as the transfer is processed.

#### Runtime Metadata

The Front-End Template exposes many helpful features and you should explore all of them while you're connected to a local development node. One good way to get started is by clicking the "Show Metadata" button at the top of the template page and reviewing [the metadata that your runtime exposes](https://substrate.dev/docs/en/knowledgebase/runtime/metadata).

![Metadata JSON](https://substrate.dev/docs/assets/tutorials/first-chain/metadata.png)

#### Pallet Interactor & Events

You can use the runtime metadata to discover a runtime's capabilities. The Front-End Template provides a helpful Pallet Interactor component that provides several mechanisms for interacting with a Substrate runtime.

![Pallet Interactor &amp; Events](https://substrate.dev/docs/assets/tutorials/first-chain/interactor-events.png)

[Extrinsics](https://substrate.dev/docs/en/knowledgebase/learn-substrate/extrinsics) are the runtime's callable functions; if you are already familiar with blockchain concepts, you can think of them as transactions for now. The Pallet Interactor allows you to submit [unsigned](https://substrate.dev/docs/en/knowledgebase/learn-substrate/extrinsics#unsigned-transactions) or [signed](https://substrate.dev/docs/en/knowledgebase/learn-substrate/extrinsics#signed-transactions) extrinsics and also provides a button that makes it easy to invoke an extrinsic by way of [the `sudo` function from the Sudo pallet](https://substrate.dev/rustdocs/v3.0.0/pallet_sudo/enum.Call.html#variant.sudo). You can learn more about using the "SUDO" button to invoke privileged extrinsics in the [Add a Pallet](https://substrate.dev/docs/en/tutorials/add-a-pallet) tutorial.

You can select Query interactions to read [the values present in your runtime's storage](https://substrate.dev/docs/en/knowledgebase/runtime/storage). The RPC and Constant options provide additional mechanisms for runtime interaction.

Like many blockchains, Substrate chains use [events](https://substrate.dev/docs/en/knowledgebase/runtime/events) to report the results of asynchronous operations. If you have already used the Front-End Template to perform a balance transfer as described above, you should see an event for the transfer in the Event component next to the Pallet Interactor.

# Next Steps

🎉 Congratulations!!! 🎉

You have launched a working Substrate-based blockchain, attached a user interface to that chain, and made token transfers among users. We hope you'll continue learning about Substrate!

Your next step may be:

* Extend the features of the template node in the [Add a Pallet](https://substrate.dev/docs/en/tutorials/add-a-pallet) tutorial.
* Learn about forkless runtime upgrades in the [Forkless Upgrade a Chain](https://substrate.dev/docs/en/tutorials/forkless-upgrade) tutorial.

If you experienced any issues with this tutorial or want to provide feedback, You can [ask a question on Stack Overflow](https://stackoverflow.com/questions/tagged/substrate) and use the `substrate` tag or contact us on [Element](https://matrix.to/#/#substrate-technical:matrix.org).


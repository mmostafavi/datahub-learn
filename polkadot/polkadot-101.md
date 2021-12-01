# What is Polkadot?

Polkadot enables scalability by allowing specialized blockchains to communicate with each other in a secure, trust-free environment.

Polkadot is built to connect and secure unique blockchains, whether they be public, permission-less networks, private consortium chains, or oracles and other Web3 technologies. It enables an internet where independent blockchains can exchange information under common security guarantees.

Polkadot is a living network with the core pillars of governance and upgradeability. The network has an advanced suite of governance tools and, using the [WebAssembly](https://webassembly.org/) standard as a "meta-protocol", can autonomously deploy network upgrades. Polkadot adapts to your growing needs without the risks of network forks.

By connecting these dots, Polkadot serves as a foundational part of a decentralized web, where users control their data and are not limited by trust bounds within the network.

## Why build in the Polkadot ecosystem?

Polkadot is a sharded blockchain, meaning it connects several chains together in a single network, allowing them to process transactions in parallel and exchange data between chains with security guarantees.

Developers can build application-specific chains using [Substrate](https://www.substrate.io/) optimized for their unique use case. Here are the advantages of the Substrate framework and the Polkadot ecosystem:

1. Native Polkadot compatibility: Any blockchain built with Substrate will be natively compatible with Polkadot, so when the mainnet comes you can connect to Polkadot as a parachain.
2. Interchain Connectivity: By connecting your blockchain to Polkadot, your blockchain will be able to pass arbitrary messages to other chains in the Polkadot network.
3. Fork-free upgrades: Upgrade your blockchain without a hard fork. Your runtime is a Wasm binary stored on-chain and can be updated using your chain’s governance mechanism.
4. Instant security: Simply by connecting your blockchain to Polkadot, your blockchain will be secured by Polkadot's [pooled security](https://medium.com/polkadot-network/how-polkadot-tackles-the-biggest-problems-facing-blockchain-innovators-1affc1309b0f).
5. Plug-and-play modular framework: Substrate allows you to simply plug in the functionalities you need, while also giving you the freedom to customize as needed.
6. Multiple Languages:** With Substrate, you can write your blockchain logic in any language that can compile to WebAssembly (Rust, C/C++, C#, Go, etc).

## What can you build on Polkadot?

Substrate comes with everything you need to build your blockchain. Use Substrate’s pallets to easily create what you want, or craft your own custom logic. The Polkadot ecosystem already hosts over 100 application-specific blockchains in development, focus on smart contracts, DeFi, IoT, governance, identity, privacy, and more.

# Network Specifications

## Transaction Fees

Polkadot uses a weight-based fee model as opposed to a gas-metering model. As such, fees are charged prior to transaction execution; once the fee is paid (in DOT), nodes will execute the transaction.

Fees on the Polkadot Relay Chain are calculated based on three parameters:

* A per-byte fee (also known as the "length fee")
* A weight fee
* A tip (optional)

The length fee is the product of a constant per-byte fee and the size of the transaction in bytes.

## Transaction Speed & Finality

While custom consensus mechanics can be implemented for your application-specific blockchain, Polkadot's finality protocol finalizes batches of blocks based on availability and validity checks that happen as the proposed chain grows.

The time to finality varies with the number of checks that need to be performed (and invalidity reports cause the protocol to require extra checks). The expected time to finality is 12-60 seconds on the relay chain.

## Languages supported

Polkadot has [implementations in various programming languages](https://wiki.polkadot.network/docs/en/learn-implementations) ranging from Rust to JavaScript. Currently, the leading implementation is built in Rust and built using the Substrate framework.

With Substrate, you can write your blockchain logic in any language that can compile to WebAssembly (Rust, C/C++, C#, Go, etc).

## EVM compatibility

Substrate EVM will allow Substrate-based blockchains, including Polkadot parachains, to host a nearly-complete instance of the Ethereum state transition function on-chain, alongside any additional Substrate modules as required for custom functionality.

Existing Solidity applications can be deployed and executed in this environment, and will gain the added benefits of being part of a Substrate-based blockchain.

## Role of the DOT token

DOT serve three key functions in Polkadot:

* to be used for governance of the network,
* to be staked for operation of the network,
* to be bonded to connect a chain to Polkadot as a parachain.

DOT can also serve ancillary functions by virtue of being a transferrable token. For example, DOT stored in the Treasury can be sent to teams working on relevant projects for the Polkadot network.

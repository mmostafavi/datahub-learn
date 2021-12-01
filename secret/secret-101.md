{% hint style="info" %}
Feel Free to skip this section and [**start the challenges**](https://learn.figment.io/tutorials/setup-the-secret-project)
{% endhint %}

------------------------

# What is Secret?

Secret Network is the first blockchain with privacy-preserving smart contracts. That means applications built on Secret can utilize encrypted data without revealing it to anyone, even the nodes in the network. For the first time, Secret Network allows developers to build powerful, permissionless, privacy-preserving applications - Secret Apps.

Secret Apps - powered by “secret”, privacy-preserving smart contracts - are built with CosmWasm smart contracts that are written in the Rust programming language and deployed on the Secret Network.

The Secret Network blockchain itself is based on Cosmos SDK / Tendermint, meaning the network has its own independent consensus, on-chain governance, and features like slashing and delegation.

## Why build on Secret?

Secret Network combines the best features of decentralized, open-source networks, and blockchains with the benefits of data privacy and improved usability. These improvements are critical for achieving mass adoption.

Developers can enjoy the benefits of smart contracts and DApps: they are robust, unstoppable, censor-resistant, and transparent. But blockchains, and by extension smart contracts have one glaring problem that is often overlooked - all data stored on them is public. In that sense, blockchains are worse than existing Web 2 solutions when it comes to privacy.

Instead of trusting your data with a single organization, you now have to trust everyone. Secret contracts and Secret Apps combine these benefits of DApps but add the ability to keep data private, helping remove one of the biggest barriers to mass adoption of the decentralized web.

## What can you build on Secret?

As the first blockchain to support encrypted inputs, encrypted outputs, and encrypted state for smart contracts, Secret Network allows for new kinds of powerful decentralized applications to be built. This can unlock substantial value across multiple verticals and industries, including the following:

* Decentralized Finance: Support for privacy-preserving asset transfers, lending, and more
* Data sharing: Sharing and monetizing data while keeping it secure
* Machine learning: Discovering powerful insights while protecting underlying data
* Healthcare: Allowing research over encrypted data, driving innovation
* Gaming Industry: Enabling more open and more usable gaming experiences

# Network Specifications

## Transaction Fees

Transaction fees are currently paid in the native token, SCRT. Since the Secret Network can handle many transactions quickly and the demand for transactions is low, the current cost of Secret transactions is very low (ie. almost nothing).

## Transaction Speed & Finality

Tendermint BFT typically have block times between 1 and 8 seconds, and can handle up to thousands of transactions per second. The Secret Network has a block time of about 7.25 seconds per block.

A property of the Tendermint consensus algorithm is instant finality. This means that forks are never created as long as more than a third of the validators are honest [**Byzantine**](https://en.wikipedia.org/wiki/Byzantine_fault). Users can be sure that their transactions are finalized as soon as a block is created, unlike Proof-of-Work blockchains, like Bitcoin and Ethereum.

## Languages supported

Secret Contracts are developed with the Rust programming language and compiled to WebAssembly ([WASM](https://rustwasm.github.io/docs/book/)) binaries.

## EVM compatibility

The Secret Network has launched a bridge with Ethereum, bringing cheaper and more usable privacy to Ethereum and Ethereum-based assets. It allows users to create synthetic (wrapped) ETH and ERC-20 tokens on Secret Network that can be used with full privacy, at a lower cost.

The bridge is bidirectional, so Secret Tokens can then be redeemed for their Ethereum-based equivalent.

## Role of the SCRT token

The SCRT is the token that: 
1. secures the Secret Network as the asset at stake, which earns its stakers new issuance rewards and transaction fees.
2. is used to pay for transactions on the network.
3. is used by stakers to vote on-chain to set Secret Network policy.

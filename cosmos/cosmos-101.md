---
description: Learn how Cosmos works and what makes it special
---

# ✏ Cosmos 101

## **What is Cosmos?**

The Cosmos SDK and Tendermint consensus provide tools that make it easy to build a new, custom-designed blockchain that will soon be interoperable with an arbitrary number of other blockchains in the Cosmos network. Chains built on Cosmos have fast, low-cost transactions and these networks are scalable.

The Cosmos Network will be an interconnected network of these chains: interoperable, sovereign blockchains that each use Tendermint consensus and the Cosmos SDK. When we say interoperable, we mean that data and token value will be able to be exchanged between sovereign blockchains. We expect this interoperability to begin after the release of the upcoming Stargate upgrade using the Inter-Blockchain Communication \(IBC\) standard. The first network to upgrade will likely be the Cosmos Hub.

While the Cosmos Network can technically exist without the Cosmos Hub, we expect that the Cosmos Hub will be the center of the Cosmos universe by facilitating secure interconnectedness, providing liquidity, collateralization, security, and more. The Cosmos Hub itself is secured by a top-in-class set of validators that validate network transactions, and these validators are bonded by the Hub’s native digital asset, the ATOM. The Cosmos Hub’s policies are set by voters whose voting power is proportional to the ATOMs that they have bonded \(ie. staked\).

### **Why build on Cosmos?**

The Cosmos SDK is the most advanced framework for building custom application-specific blockchains today. Here are a few reasons why you might want to consider building your decentralised application with the Cosmos SDK:

* The default consensus engine available within the SDK is [Tendermint Core](https://github.com/tendermint/tendermint). Tendermint is the most \(and only\) mature BFT consensus engine in existence. It is widely used across the industry and is considered the gold standard consensus engine for building Proof-of-Stake systems. _\*\*_
* The SDK is open source and designed to make it easy to build blockchains out of composable [modules](https://docs.cosmos.network/v0.39/x/). As the ecosystem of open source SDK modules grows, it will become increasingly easier to build complex decentralised platforms with it. 
* The SDK is inspired by capabilities-based security, and informed by years of wrestling with blockchain state-machines. This makes the Cosmos SDK a very secure environment to build blockchains. 
* Most importantly, the Cosmos SDK has already been used to build many application-specific blockchains that are already in production. Among others, we can cite [Cosmos Hub](https://hub.cosmos.network/), [IRIS Hub](https://irisnet.org/), [Binance Chain](https://docs.binance.org/), [Terra](https://terra.money/) or [Lino](https://lino.network/). [Many more](https://cosmos.network/ecosystem) are building on the Cosmos SDK.

### **What can you build on Cosmos?**

The Cosmos SDK allows any developer to build application-specific blockchains. Application-specific blockchains are blockchains customized to operate a single application. Instead of building a decentralised application on top of an underlying blockchain like Ethereum, developers build their own blockchain from the ground up. This means building a full-node client, a light-client, and all the necessary interfaces \(CLI, REST, ...\) to interract with the nodes. Here are some benefits of building your application-specific blockhain:

* Build a blockchain with the state-machine in the programming language of your choice 
* Swap the consensus engine to what matches your needs \(today only Tendermint is production-ready\)  
* Select the tradeoffs \(e.g. number of validators vs transaction throughput, safety vs availability in asynchrony, ...\) and design choices \(DB or IAVL tree for storage, UTXO or account model, ...\) that work for you  
* Optimize performance as your application-specific blockchain only operates a single application instead of competing for computation and storage  
* Use off the shelf cryptography or your own while relying on well-audited crypto libraries  
* Configure the level of decentralization that is right for you 

Check out other projects building on the Cosmos SDK [here](https://cosmos.network/ecosystem).

## **Network Specifications**

### **Transaction Fees**

Transaction fees are currently paid in the native token, the ATOM, but in the near future the Cosmos Hub will be able to accept fees in other tokens via[ Inter-Blockchain Communication \(IBC\).](https://figment.io/resources/inter-blockchain-communication-ibc-is-coming-to-cosmos/) Since the Cosmos Hub can handle many transactions quickly and the demand for transactions is low, the current cost of Hub transactions is very low \(ie. almost nothing\).

### **Transaction Speed & Finality**

Tendermint BFT typically have block times between 1 and 8 seconds, and can handle up to thousands of transactions per second. The Cosmos Hub has a block time of about 7.25 seconds per block.

A property of the Tendermint consensus algorithm is instant finality. This means that forks are never created as long as more than a third of the validators are honest \([Byzantine](https://en.wikipedia.org/wiki/Byzantine_fault)\). Users can be sure that their transactions are finalized as soon as a block is created, unlike Proof-of-Work blockchains, like Bitcoin and Ethereum.

### **Languages supported**

The Cosmos SDK supports the Go language so you can take advantage of the Golang ecosystem to build your DApp.

### **EVM compatibility**

While there is no EVM module, the upcoming [Ethermint chain](https://ethermint.zone/) will be a scalable and interoperable Ethereum, and we expect that the Cosmos Hub chains will be interoperable with Ethermint using[ Inter-Blockchain Communication \(IBC\).](https://figment.io/resources/inter-blockchain-communication-ibc-is-coming-to-cosmos/)

### **Role of the ATOM token**

The ATOM is the token that 1\) secures the Cosmos Hub as the asset at stake, which earns its stakers new issuance rewards and transaction fees, 2\) is used to pay for transactions on the network, and 3\) is used by stakers to vote on-chain to set Cosmos Hub policy.


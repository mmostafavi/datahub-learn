---
description: Learn how Mina works and what makes it special
---

# ✏ Mina 101

## **What is Mina?**

Mina is the first cryptocurrency protocol with a succinct blockchain. Current cryptocurrencies like Bitcoin and Ethereum store hundreds of gigabytes of data, and as time goes on, their blockchains will only increase in size. With Mina however, no matter how much the usage grows, the blockchain always stays the same size - about 22kb \[1\] \(the size of a few tweets\). This means participants can quickly sync and verify the network.

This breakthrough is made possible due to zk-SNARKs - a type of succinct cryptographic proof. Each time a Mina node produces a new block, it also generates a SNARK proof verifying that the block was valid. All nodes can then store the small proof, as opposed to the entire chain. By not having to worry about block size, the Mina protocol enables a blockchain that is decentralized at scale.

### **Why build on Mina?**

A fixed-size blockchain gives Mina unparalleled censorship resistance, enabling **fully censorship resistant money.** Mina is the first fully censorship resistant **medium of exchange** that’s built for everyday use _and_ built to integrate into traditional services. A vibrant permissionless economy needs a **store of value** people are willing to spend — a behavior naturally disincentivized by the prospect of transacting in a scarce fixed-supply asset like bitcoin.

zk-SNARKs give Mina **permissionless apps** that compute logic and data off-chain for privacy and scalability, verifying them later on-chain for integrity.

Mina’s protocol design gives developers **easy access to digital dollars** — anyone building on Mina can integrate stablecoin payments into their apps with just a couple lines of code.

### **What can you build on Mina?**

As the world’s lightest blockchain, Mina enables an entirely new category of applications called Snapps \[3\] : Snarkified Applications. Snapps are featurewise similar to Dapps on Ethereum, but are superior thanks to three specific properties:

1. Verify the integrity of a piece of data without disclosing what it is.
2. Verify correct execution of expensive computations.
3. Significant scalability benefits.

These types of applications aren’t necessarily new - in a way both exist as present day blockchains: ZCash as a Snapp with property 1, and Mina as a Snapp with property 2.

A Snapp is also significantly more efficient than a Dapp on Ethereum. In order for the Ethereum world computer to make a commitment to its users about the execution of a Dapp, every node and miner on the network has to run the same computation. This is wildly inefficient. With a Snapp on Mina, the Snapp gets executed once by its developer, after which all other nodes can just verify the associated SNARK proof. One can make the same argument for SNARK powered Layer 2 Dapps on Ethereum, however those Dapps are still encumbered by the limited throughput of the main chain, whereas a Snapp on Mina benefits from the scalability potential of the Mina blockchain thanks to its succinct nature.

In general, a Snapp on Mina has the following workflow:

1. Identify the code to run, open source it if it isn’t already
2. Use the code to deploy a SNARK circuit via a function call on Mina
3. Source the data to perform the computation on
4. Call the function with the relevant data
5. Computation runs on chain, similar to a smart contract \[ready 6-12 months after mainnet\]
6. Attach the SNARK proof returned from the function to a Mina address
7. Mina executes transactions based on result of the SNARK proof

## **Network Specifications**

### **Transaction Fees**

The average gas price suggested on the Mina protocol is 0.001 MINA.

### **Transaction Speed & Finality**

With each subsequent block, a recipient has a higher degree of confidence that the payment is actually complete and that the network has consensus about that block. However, like in most blockchains, payments are said to be confirmed after a certain number of blocks, also known as transaction finality.

In the Bitcoin network, a transaction is confirmed after [6 blocks](https://en.bitcoin.it/wiki/Confirmation) \(60 mins\) with an assumption that an attacker is unlikely to amass more than 10% of the hashrate.

With slot duration of 3 mins and assuming 90% honest stake, the following table shows the finality in blocks, average time it takes to produce the corresponding number of blocks and the confidence that payment will be confirmed.

| Finality \(in blocks\) | Average time for finality | Finality confidence \(%\) |
| :--- | :--- | :--- |
| 8 | 33 mins | 98.6709 |
| 15 | 60 mins | 99.9231 |
| 23 | 1hr 32mins | 99.9965 |
| 30 | 2hrs | 99.9998 |
| 38 | 2hrs 32mins | 100 |

Average time is calculated based on consensus constants that determine the number of slots filled per epoch. This is currentlyset to 75%

The recommended wait time for a transaction to be confirmed is 15 blocks which provides a 99.9% confidence that the transaction will not be reversed.

### **Languages supported**

Mina currently supports Graphql and is planning to add support for Javascript as a development language for Snapps.

### **EVM compatibility**

While Mina does not currently support the EVM, the Ethereum Foundation and Mina Foundation have announced a new RFP for the design and implementation of a mechanism to verify the Pickles SNARK on Ethereum.

The goal of this is \(1\) to enable full-verification of the Mina blockchain on Ethereum to enable inter-op between the two chains, and \(2\) to enable applications more generally to use recursive SNARKs on Ethereum.

### **Role of the MINA token**

The Mina network’s native asset lies at the center of the protocol’s economic incentive system. Block producers and snark producers are paid in MINA, and all holders are incentivized to stake the token: staking rewards during Mina’s first year post-launch will be worth up to 24% of staked capital, with further emissions \_\*\*\_rewards thereafter.


# What is Polygon?

{% hint style="info" %}
"_Polygon is a protocol and a framework for building and connecting Ethereum compatible blockchain networks. Aggregating scalable solutions on Ethereum, supporting a multi-chain Ethereum ecosystem._"
{% endhint %}

**Polygon** is designed to interoperate with and solve some of Ethereum's limitations: Scaling, the speed of transaction throughput - and poor user experience such as the high cost of transactions. Polygon's testnets can currently reach over 7000 transactions per second and the low cost of the MATIC token means cheap transaction fees!  
  
Matic, the predecessor of **Polygon**, utilized a single scaling solution known as [Plasma](https://education.district0x.io/general-topics/understanding-ethereum/understanding-plasma/) which allows for 'child' blockchains to move transaction bloat off of the main Ethereum network. A Plasma chain can have different conditions to the main network, and can be specifically tuned to satisfy certain needs. Sidechains based on Plasma offer cheap and rapid transactions that are ultimately settled in batches on the main chain, so they benefit from the security of the main chain without compromising performance.

Today, **Polygon** operates a Proof-of-Stake (PoS) sidechain. **Polygon** will grow to include other scaling solutions like [zero knowledge proofs](https://consensys.net/blog/blockchain-explained/zero-knowledge-proofs-starks-vs-snarks/) and [optimistic rollups](https://blog.polygon.technology/polygon-research-ethereum-scaling-with-rollups-8a2c221bf644) in the future, and allow developers to choose how best to scale their dApps. Currently, connecting to Polygon purely from a user perspective means using a compatible wallet such as Metamask, and being aware of the difference between Polygon and Ethereum.

[**MATIC**](https://coinmarketcap.com/currencies/polygon/) is the native token of **Polygon**, as [ETH](https://coinmarketcap.com/currencies/ethereum/) is the native token of Ethereum. MATIC is used to pay transaction (gas) fees on **Polygon**. It is also used to secure the network via staking and to pay out staking rewards. Holding MATIC allows users to take part in the decentralized governance of the protocol by voting on [Polygon Improvement Proposals](https://forum.matic.network/t/polygon-improvement-proposals/630).

Check out [Awesome Polygon](https://awesomepolygon.com/dapps/) for many examples of what is being built on **Polygon**! [This article](https://finematics.com/polygon-commit-chain-explained/) on Finematics is also worth a look.

There are a few important sites that will be of interest to users of **Polygon**

* [For bridging assets between **Polygon** and Ethereum](https://wallet.matic.network)
* [For knowledge!](https://docs.matic.network/) 
* [For free testnet assets](https://faucet.matic.network/)

-------------------------------------

## Heimdall & Bor

For **Polygon**, the node is designed with a two layer implementation with Heimdall (the Validator layer) & Bor (the Block Producer layer) :

* **Heimdall** uses Proof-of-Stake verification and is responsible for checkpoint blocks on Ethereum mainnet. It also handles the Validators and rewards management. The role of Validators is to run a full node, produce blocks, validate and participate in consensus and commit checkpoints on the Ethereum Mainnet. Heimdall ensures synchronization with Ethereum.
  * On DataHub, the RPC endpoint is used to access Heimdall. 
* **Bor** is compatible with the Ethereum Virtual Machine (EVM), it is a fork of the [Geth client](https://geth.ethereum.org/docs/) with customized consensus algorithms. User interaction on **Polygon** takes place on this chain, it also makes available the functionality and compatibility of Ethereum developer tooling and applications.  Read more about the [core concepts](https://docs.matic.network/docs/contribute/bor/core_concepts) of Bor.
  * On DataHub, the JSON-RPC endpoint is used to access Bor.

-------------------------------------

## Bridging Polygon & Ethereum

Polygon's Validators continuously monitor a contract on Ethereum's mainnet called _**StateSender**_. Each time a registered contract on Ethereum calls this contract, it emits an event. Using this event, Polygon validators relay the data to another contract on the **Polygon** chain. This _**StateSync**_ mechanism is used to send data from Ethereum to **Polygon**.

Validators also periodically submit a hash of all transactions on the **Polygon** chain to the Ethereum chain. This is the mechanism which enables two way data \(state\) transfer between **Polygon** and Ethereum. The [_**Checkpoint**_](https://docs.matic.network/docs/contribute/heimdall/checkpoint) can be used to verify any transaction that happened on **Polygon**. Once a transaction is verified to have happened on **Polygon** chain, action can be taken accordingly on Ethereum. 

It is possible to process many transactions on **Polygon** and use relatively little MATIC for gas fees. This helps reduce friction for onboarding new users, because there is a much lower barrier to entry in terms of raw cost as compared to settling the same number of transactions on Ethereum.

In summary, **Polygon** is an attractive solution for developers who wish to keep much of the tooling and community of Ethereum, while leveraging the best scaling for their decentralized applications.

-------------------------------------

# Next Steps

Now that we gave you a quick overview of Polygon ecosystem, we still need to set up our environment.

{% hint style="info" %}
Feel Free to skip this section and [start the challenges](https://learn.figment.io/tutorials/setup-the-near-project)
{% endhint %}

------------------------

# What is NEAR?

NEAR is a decentralized application platform that makes blockchain accessible to everyday people who are excited for the future of the Open Web. A web that is more private, more secure, and puts people in control of their money, data, and identity. NEAR’s purpose is to enable community-driven innovation to benefit people around the world.

What makes NEAR special?

1. NEAR is the inevitable evolution of existing platforms 
   1. Bitcoin made digital currency a reality, a global transfer of value programmed on a blockchain. Ethereum took it a step further and made it possible to create basic applications leveraging digital currency on blockchains, but has faced issues with scaling and prohibitive fees.  
   2. NEAR solves these challenges and finally makes building the Open Web possible by introducing a breakthrough sharding design that creates opportunities for innovation never possible in blockchain before. 
2. NEAR is made for the real world 
   1. NEAR is the fastest path to market for builders to create blockchain applications that are as easy to use as they are to build. It uses common development patterns and programming languages that are immediately familiar to web developers. 
   2. NEAR was built with pragmatism of a Silicon Valley tech startup by engineers who have shipped scalable solutions for the world’s leading technology companies. 
   3. NEAR is secure enough for high value assets like Digital Currencies or Identities and performant enough to scale for DeFi or Gaming applications without prohibitively expensive fees or network latency. 
3. NEAR is designed to be interoperable 
   1. NEAR is designed to work with existing blockchain platforms and is easy for existing applications to migrate to. NEAR uses the Ethereum Virtual Machine and Web Assembly so that builders can seamlessly swap from Ethereum to NEAR (and back) on the NEAR Rainbow Bridge to leverage the performance, security, and favorable economics of NEAR without having to rebuild their applications or smart contracts. 
   2. With NEAR, you don’t have to choose between NEAR and Ethereum or commit to only one blockchain. Interoperability and usability across platforms will be key to the mass adoption of blockchain.

NEAR is the first blockchain platform truly built for the real world. It is a builder's fastest path to market to create blockchain applications that are as easy to use as they are to build.

# Why build on NEAR?

**NEAR is easy to learn**. It uses familiar development patterns and languages for portability and ease of access for any web developers.

* “Writing smart contracts on Rust is awesome. It allows you to code error-free and fault-free. It’s a big reason why we chose NEAR. Other blockchains require you to learn less popular programming languages.” - _Carolin Wend, Co-Founder of Mintbase_

**NEAR is versatile**. It's secure enough for high-value assets like money or identity, and performant enough to scale DeFi or Gaming apps. NEAR helps developers avoid high fees and slow speeds to deliver a delightful user experience.

* “Augur gets $1B in volume and users burn $63M in gas fees. Flux gets $1B in volume and users burn only $10k in gas fees.” _Peter Mitchell, CEO of Flux_

**NEAR is by builders for builders**. NEAR is built by a pragmatic team of engineers that have shipped scalable solutions for many of the world’s leading tech companies. They’ve earned the backing of leading investors including Coinbase Ventures, Andressen Horowitz, Pantera Capital and Naval Ravikant.

* “NEAR approached us with a developer-first attitude hellbent on helping creators create seamless and memorable user experiences. We intuitively felt that NEAR could be the silver bullet that ZEST needed.” _Geoff Wellman, Founding Partner, CTO VHS_

**NEAR works with existing blockchain platforms.** Builders can seamlessly swap from Ethereum to NEAR (and back) on the NEAR Rainbow Bridge to leverage the performance, security, and favorable economics of NEAR without having to rebuild their applications. With NEAR, there is no need to choose between chains. Builders and users get the best of both worlds.

**NEAR makes applications easy to use**. In some cases, users don’t realize they’re using a blockchain application. NEAR Accounts combine human-readable names and a flexible contract-based permission structure that enables business models like recurring, free to play, or freemium that were never before possible using blockchain applications.

**NEAR is fast and future proof**. The unique sharding techniques implemented to enable the NEAR protocol to maintain consensus while splitting the network into multiple pieces. This means that throughput and speed will increase exponentially as the number of nodes on the network increases. There is no theoretical limit on the NEAR network’s capacity. As the Open Web grows, NEAR scales to meet it.

# What can you build on NEAR?

1. Open (decentralized) Finance: NEAR provides both a synchronous environment for atomic transactions, which benefits fast-moving financial applications like decentralized exchanges, and an asynchronous environment which allows every other application to hook into these critical financial rails. This will be possible with the upcoming EVM support.  
2. Open Web: because NEAR is basically the “builder’s blockchain”, it has been optimized to provide substantial flexibility and performance for developers to build great experiences for their end users. This means that consumer-facing blockchain-backed apps are finally possible and the use cases of the Open Web can finally be realized: 
   1. Prediction markets -- markets like Augur have been handicapped by the difficulty of building smooth user experiences on Ethereum.  Apps like [Flux](https://flux.market) on NEAR provide much more user-friendly ways to set up programmatically enforceable bets with fast, easy payments. 
   2. NFT marketplaces -- marketplaces like [Mintbase](https://mintbase.io), which transferred from Ethereum, allow creators of unique digital assets to easily implement marketplaces to buy and sell them. On NEAR, they have access to reasonable costs plus intuitive end-user experiences. 
   3. Gaming -- because NEAR allows for performant and usable apps, developers can create gaming experiences that hook into the blockchain without forcing end-users to suffer through wallets or tokens.

[Take a look at these examples from a recent hackathon](https://near.org/blog/winners-of-hack-the-rainbow)

# Network Specifications

## Transaction Fees

First, transaction fees on NEAR are a tiny fraction of the amount of fees paid on other blockchains. 1000 times lower transaction fees than Ethereum for users and developers earn 30% of transaction fees.

NEAR keeps transaction fees low by dynamically resharding to optimize for performance and avoid congestion on the network. However, the most important feature of the NEAR token isn’t related to fees or rewards, it comes back to the usability and developer experience that truly sets NEAR apart.

Complicated blockchain concepts like tokens, rewards, and fees can be almost entirely hidden from end-users of NEAR applications because smart contracts can be configured to pay fees on their behalf. This allows developers to build experiences which feel more like today’s web to their end-users, but still take advantage of the underlying innovations of blockchain.

## Transaction Speed & Finality

The expected block time is around 1s and the expected time to finality is around 2s. 1,000 TPS benchmarked on a single shard for token transfers (contract ops are slower).

## Languages supported

* **Contracts**: Rust and AssemblyScript
* **Command line**: NEAR CLI
* **API**: JavaScript and JSON RPC API

## EVM compatibility

NEAR is fully compatible with the EVM and interoperable with Ethereum. NEAR does not want Ethereum developers to choose between the two networks and commit to only one. Developers can have the same asset on both blockchains and even have apps that seamlessly communicate across the boundary.

This is made possible by the [**Rainbow Bridge**](https://github.com/near/rainbow-bridge), which connects the Ethereum and NEAR blockchains by leveraging the Ethereum miners and NEAR validators. More info can be found [**here**](https://github.com/near/near-evm).

## Role of the NEAR token

The NEAR token is how people who use applications on the network pay to submit transactions to the nodes who actually run the network. The token is thus a utility — if you hold it, you can use applications hosted on the network.

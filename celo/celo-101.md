# What is Celo?

Celo is a smart contract blockchain network. The technology uses a phone-number-based identity system with address-based encryption and [eigentrust-based reputation](https://en.wikipedia.org/wiki/EigenTrust). Their first application is a social payments system that can be used on a smartphone.

Celo is preparing to launch a variety of stable coins pegged to global fiat and crypto currencies, the first one being pegged to the US dollar (cUSD).

# Why build on Celo?

The Celo blockchain is designed to empower anyone with a smartphone anywhere in the world to have access to financial services, send money to phone numbers, and pay merchants. The project aims to be a decentralized platform that is not controlled by any single entity, but instead developed, upgraded and operated by a broad community of individuals, organizations and partners.

Celo is oriented around providing the simplest possible experience for end users, who may have no familiarity with cryptocurrencies, and may be using low cost devices with limited connectivity. To achieve this, the project takes a full-stack approach, comprising of both a protocol and applications that use that protocol. Celo offers a full suite of powerful features:

* Stable value currencies (cUSD)
* Accounts linked to phone numbers
* Transaction fees in any currency
* Immediate syncing
* Programmable (full EVM compatibility)
* Carbon Neutral

# What can you build on Celo?

Celo enables developers to build a new financial system to empower their users. Developers can:
* Create financial services applications for the underbanked (savings accounts, money transfer interfaces, merchant services, lending, trading)
* Integrate with existing blockchain and traditional financial services
* Create transparent aid services
* Build payroll integrations to allow internet users to earn stable currencies
* Create advanced user wallets (smart contract wallets) 
* Alternative currencies 
* Use the full suite of Ethereum smart contracts right off the shelf

You can view some of the projects that are building on Celo [here](https://docs.celo.org/developer-guide/overview/celo-dapp-gallery).

# Network Specifications

## Transaction Fees

* [Celo uses a gas market based on EIP1559 to set gas prices](https://docs.celo.org/celo-codebase/protocol/transactions/gas-pricing)
* [There are also gateway fees for light clients](https://docs.celo.org/celo-codebase/protocol/transactions/full-node-incentives)
* Currently fees on the network are around $0.001/transaction

## Transaction Speed & Finality

Celo has a 5 seconds block time with instant finality.

## Languages supported

Most of the protocol code is written in Go, while the SDKs are written in Typescript which supports native javascript.

The Celo community is also working on Java and Python SDKs.

## EVM compatibility

While the Celo client originated as a fork of Ethereum Go langauge client, [go-ethereum](https://github.com/ethereum/go-ethereum) (or geth), it has several significant differences, including a proof-of-stake based PBFT consensus mechanism. All the cryptoassets on Celo have ERC-20 compliant interfaces, meaning that while they are not ERC-20 tokens on the Ethereum Mainnet, all familiar tooling and code that support ERC-20 tokens can be easily adapted for Celo assets, including the Celo Native Asset (CELO) and the Celo Dollar (cUSD).

In terms of programmability, Celo is similar to Ethereum. Both networks run the Ethereum Virtual Machine (EVM) to support smart contract functionality. This means that all programming languages, developer tooling and standards that target the EVM are relevant for both Celo and Ethereum. Developers building on Celo can write smart contracts in [Solidity](https://solidity.readthedocs.io/en/latest/), use [Truffle](https://www.trufflesuite.com/) for smart contract management and take advantage of smart contract standards that have already been developed for Ethereum.

## Role of the CELO token

The native Celo token (CELO) is the utility token on the Celo network. CELO are used to pay for transactions and are also staked by token-holders to earn rewards, participate in governance, and to vote for validators.


# What is Avalanche?

Avalanche is a layer one protocol by Ava Labs that offers high-throughput, fast finality, and unprecedented decentralization. Developers are able to launch their own public or private blockchains (called subnets), create and trade digital assets, and build scalable smart contracts and decentralized applications.

Subnets within the Avalanche network will be highly customizable and will have their own incentive structures.

Because of this customization, companies can create legally compliant permissioned subnets while still being connected to the grander Avalanche network. This could lead to the creation of new financial primitives and bring more traditional financial products into the blockchain space in a decentralized way.

## Why build on Avalanche?

AVA was created to fill use cases that existing platforms are unable to provide while retaining the promise of decentralization. People who use Avalanche do so because they need to support their high-throughput systems but do not wish to rely on a trusted intermediary. With the ability to partition private data away from the rest of the Avalanche network while still being a member of the overall network, Avalanche enables the privacy use cases other platforms do not.

With fast decision times and reasonable hardware requirements, Avalanche powers a system that can accept many times the data and productive yield than existing decentralized systems provide at a low cost to the end user. Avalanche is an asset engine that can deploy and power new markets or integrate existing ones into a global exchange backbone.

## What can you build on Avalanche?

Avalanche provides developers with one of the most versatile protocols available today. Developers can:

* Create their own VM for their needs
* Create their subnets (private and public), to customize their value proposition 
* Create as many chains as needed under their subnet
* Use (almost) the full suite of Ethereum smart contracts right off the shelf 
* Create native NFTs as first-order objects
* Create powerful decentralized financial applications

And more!

# Network Specifications

## Transaction Fees

In order to prevent spam, transactions on the Avalanche network require the payment of a transaction fee. The fee is paid in AVAX, Avalanche’s native token.

The transaction fee is burned. That is, the AVAX tokens paid in a transaction fee do not go to anywhere. Instead, they are destroyed forever.

When you issue a transaction through Avalanche’s API, the transaction fee is automatically deducted from one of the addresses you control.

Different types of transactions require payment of a different transaction fee. This table shows the transaction fee schedule:

```text
+----------+-------------------+------------------------+
| Chain    : Transaction Type  | Transaction Fee (AVAX) |
+----------+-------------------+------------------------+
| P        : Create Blockchain |                   0.01 |
+----------+-------------------+------------------------+
| P        : Add Validator     |                      0 |
+----------+-------------------+------------------------+
| P        : Add Delegator     |                      0 |
+----------+-------------------+------------------------+
| P        : Create Subnet     |                   0.01 |
+----------+-------------------+------------------------+
| P        : Import AVAX       |                  0.001 |
+----------+-------------------+------------------------+
| P        : Export AVAX       |                  0.001 |
+----------+-------------------+------------------------+
| X        : Send              |                  0.001 |
+----------+-------------------+------------------------+
| X        : Create Asset      |                   0.01 |
+----------+-------------------+------------------------+
| X        : Mint Asset        |                  0.001 |
+----------+-------------------+------------------------+
| X        : Import AVAX       |                  0.001 |
+----------+-------------------+------------------------+
| X        : Export AVAX       |                  0.001 |
+----------+-------------------+------------------------+
```

The C-Chain gas price is 4.7e-7 AVAX/gas. The C-Chain gas limit is 10e8.

## Transaction Speed & Finality

Protocols in the Avalanche family are very fast. They can achieve irreversible finality in 1-2 seconds, quicker than a typical credit card transaction. They support many thousands of transactions per second, over Visa’s typical throughput.

Avalanche protocols are lightweight and sustainable. Unlike Nakamoto protocols, they use very little energy, and when there is no work to do, the system quiesces (waits in a low energy consumption state).

## Languages supported

Avalanche's C-Chain uses the standard Ethereum suite of tools.

The X-Chain and P-Chain use a Typescript library that supports native JavaScript cross-platform.

## EVM compatibility

The C-Chain is fully compatible with the EVM and provides support for any Ethereum applications as the EVM API is identical to Geth's API.

## Role of the AVAX token

AVAX is the Avalanche network's native token. All transactions on the Avalanche network require a fee denominated in AVAX.

# What is Tezos?

Tezos is a secure, smart contract blockchain platform that uses its built-in governance mechanism for protocol upgrades. Tezos is one of the first blockchain platforms to introduce a formal on-chain governance process. The Tezos network is highly decentralized with over 500 nodes distributed globally. Tezos’ smart contract language, Michelson, is designed to facilitate formal verification, a technique used to improve security by mathematically proving properties. This technique, if used properly, can help avoid costly bugs and contentious debates that follow.

# Why build on Tezos?

Tezos is an open-source platform for assets and applications that can evolve by upgrading itself. Stakeholders govern upgrades to the core protocol, including upgrades to the amendment process itself. **It benefits from three main characteristics: self-amendment, on-chain governance, and decentralized innovation.**

Self-amendment allows Tezos to upgrade itself without having to split (“fork”) the network into two different blockchains. This is important as the suggestion or expectation of a fork can divide the community, alter stakeholder incentives, and disrupt the network effects that are formed over time. Because of self-amendment, coordination and execution costs for protocol upgrades are reduced and future innovations can be seamlessly implemented.

In Tezos, all stakeholders can participate in governing the protocol. The election cycle provides a formal and systematic procedure for stakeholders to reach agreement on proposed protocol amendments. By combining this on-chain mechanism with self-amendment, Tezos can change this initial election process to adopt better governance mechanisms when they are discovered.

Proposed amendments that are accepted by stakeholders can include payment to individuals or groups that improve the protocol. This funding mechanism encourages robust participation and decentralizes the maintenance of the network. Fostering an active, open, and diverse developer ecosystem that is incentivized to contribute to the protocol will facilitate Tezos development and adoption.

# What can you build on Tezos?

Turing-complete smart contract platforms like Tezos or Ethereum allow for arbitrary code to be executed in a trust-minimized manner. However, certain applications may be well suited for Tezos based on formal governance and focus on smart contract security. Below are some preliminary examples:

1. Digital Assets

  Assets like digital money, tokenized real estate, stablecoins, digital collectibles, and so on are particularly well-suited for Tezos. Avoiding contentious forks can preserve value and coordination around one network, making Tezos a compelling platform for issuing digital assets.

Although no system can be unconditionally secure, the Tezos smart contract language, Michelson, was designed with security and formal verification in mind. This is especially critical for smart contracts representing high value assets, given the unforgiving nature of smart contracts bugs.

2. Trust-Minimized Financial Contracts

  Financial contracts such as decentralized exchanges, swaps, loans, and so on demand a high-level of correctness. Decentralized blockchain networks derive their value from the absence of a trusted third party, which makes a loss of funds from a bug in the code particularly unforgiving.

# Network Specifications

## Transaction Fees

So far, this system is similar to the way Ethereum handles transaction. However, we differ in the way we handle fees. Ethereum allows arbitrarily long programs to execute by requiring a fee that increases linearly with the program's executing time.

## Transaction Speed & Finality

Currently, Tezos does around 30-40 transactions per second.

In the current Tezos protocol, 30 confirmations (~30 minutes) may be considered a good rule of thumb for transaction to be considered final. Since Tezos uses a chain-based PoS consensus algorithm, the possibility of a chain re-organization remains after a transaction. Users must wait a number of confirmations before they can be overwhelmingly confident that a transaction will not be reversed.

## Languages supported

There are various high-level languages available for programming Smart Contracts for Tezos. The intention of these languages is to ease the development experience, so you can focus on the content of your Smart Contracts, rather than the implementation.

Current options in development include:

* LIGO: A Smart Contract language offering Pascal, Ocaml, and ReasonML syntax flavors
* SmartPy: A Python library with tools to write and test Smart Contracts
* Morley/Lorentz: A Haskell-flavored eDSL for writing Smart Contracts

## EVM compatibility

Tezos does not currently support an EVM implementation.

## Role of the XTZ token

XTZ, tez, or ꜩ is the native currency of Tezos. XTZ is programmable money created by smart contract on the Tezos blockchain. It is used to pay on-chain fees and influence the network's governance.

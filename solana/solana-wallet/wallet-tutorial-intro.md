You're about to learn how to develop a wallet for the [Solana](https://solana.com/) protocol. As you build, we'll dive into some of the fundamentals of Web 3 so you can start cultivating an understanding for blockchain technology and decentralized applications (dApps).

# Prerequisites 📋

There are no formal blockchain or Web 3 prerequisites for this tutorial, but you should have some experience with [TypeScript](https://www.typescriptlang.org/) and [React](https://reactjs.org/). Ideally you have heard about [Solana](https://solana.com/), and know that it's a public blockchain protocol capable of running smart contracts with significant scalability advantages.

Having said that, feel free to give it a shot even if you don't meet the above prerequisites since you'll likely get a lot out of it anyway. Just be extra patient because it may be a bit more difficult to navigate.

We think of ourselves as your guide - for a brief amount of time - on your journey to build a better internet. We hope you walk away with a better understanding of Web 3 technologies and perhaps even a bit more confidence to tackle significant problems.

We've tried to show you the door (or maybe part of it). Your job is to discover your path and walk it courageously.

![Figure 1: You take the red pill — you stay in Wonderland and I show you how deep the rabbit hole goes](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/cat.jpeg?raw=true)

{% label %}
Figure 1: You take the red pill — you stay in Wonderland and I show you how deep the rabbit hole goes

# System dependencies 💻

Make sure you have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Node](https://nodejs.org/en/) and [yarn](https://yarnpkg.com/getting-started/install) installed.

# Setting up 🧱 

You'll be building Solana interactions on top of a [Next.js](https://nextjs.org/) app that we have pre-built for you. This way you can skip the development phase you're familiar with (e.g. writing React components and Next pages) and dive right into the blockchain-specific phase.

First, clone [the repo](https://github.com/figment-networks/wallet-tutorial) in your system and install the app dependencies by running the following three commands in your terminal:

##### _Listing 1.1: Set up commands to run on your terminal_
```text
$ git clone https://github.com/figment-networks/wallet-tutorial.git
$ cd wallet-tutorial
$ yarn
```

Then, run `yarn dev` in your terminal to start the development server on port 3000. The terminal should print output similar to that in [Listing 1.2](#listing-12-terminal-message-on-app-start-up) informing you that the app is available to be viewed at [http://localhost:3000](http://localhost:3000).

##### _Listing 1.2: Terminal message on app start up_
```text
yarn run v1.22.11
$ yarn dev
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
info  - Using webpack 5. Reason: Enabled by default https://nextjs.org/docs/messages/webpack5
event - compiled successfully
```

# Tutorial structure 🏗 

The tutorial is structured as a set of steps that break down the process of developing the wallet functionality for the pre-built Next.js app. Each step discusses key concepts and mental models that create the necessary context for you to better understand what we're building and why. Think of this as the warm up.

After the warm up, we provide a step-by-step implementation of the functionality including a code snippet of the full solution. After building context with the warm up, this serves as a way for us to guide you through the logic of the implementation and helps scaffold your mental model for the topic.

The implementation is followed by a challenge that aims to give you just enough instructions and references to allow you to implement the functionality on your own if you want to take a shot at it. This allows you to exercise [active recall](https://en.wikipedia.org/wiki/Active_recall) and increases your mind's ability to make connections.

# Wallet preview 🔍

By the time you're done with this tutorial, you'll have a functional wallet capable of changing the state of the Solana blockchain by receiving and transferring onchain funds. In [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2), we'll generate a 12-word phrase that will grant us access to a wallet.

![](https://github.com/figment-networks/learn-tutorials/blob/master/solana/solana-wallet/assets/create.png?raw=true)

Then we'll build a function to fetch the balance of our newly created wallet in [Step 3](https://learn.figment.io/tutorials/solana-wallet-step-3) before leveraging functionality to issue "test tokens" through something called an "airdrop" that we'll explore in [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4).

![](https://github.com/figment-networks/learn-tutorials/blob/master/solana/solana-wallet/assets/airdrop.png?raw=true)

Having added funds to our wallet, we'll build the ability to transfer funds to another wallet in [Step 5](https://learn.figment.io/tutorials/solana-wallet-step-5) using bank checks as a scaffolding analogy.

![](https://github.com/figment-networks/learn-tutorials/blob/master/solana/solana-wallet/assets/transfer.png?raw=true)

Finally, we'll use the phrase we generated in [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2) to recover access to it in [Step 6](https://learn.figment.io/tutorials/solana-wallet-step-6).

![](https://github.com/figment-networks/learn-tutorials/blob/master/solana/solana-wallet/assets/import.png?raw=true)

Although building a production-ready, non-custodial wallet like [Phantom](https://phantom.app/) or [MetaMask](https://metamask.io/) is beyond the scope of this tutorial, by the end of it you'll have developed a foundation of judgment that will let you better allocate your learning time as you explore new projects. To assist with that, we'll end the tutorial with a list of [additional resources](https://learn.figment.io/tutorials/solana-wallet-conclusion#additional-resources) that you'll be well prepared to explore.

![Figure 2: A journey of a thousand miles begins with a single step](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/journey.jpeg?raw=true)

{% label %}
Figure 2: A journey of a thousand miles begins with a single step

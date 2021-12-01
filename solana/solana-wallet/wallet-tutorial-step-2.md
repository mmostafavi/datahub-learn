# Crypto gateway

Crypto wallets are the most critical piece of user-facing infrastructure in the blockchain space. In the same way that the browser brought the web to a mass audience by providing an easy-to-use gateway into the underlying servers and databases, wallets provide users with an easy-to-use gateway into blockchain protocols.

A wallet is less like the wallet you use for your credit cards, and a lot more like a keychain. If you think of a blockchain as a giant bank safe with digital safety deposit boxes, wallets are the keychains where you keep the key to your box. Once you have access to a box, you can manage the contents of it by receiving, sending, holding and spending digital assets.

![Figure 3: Wallets hold private keys to your public address on a blockchain](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/safe.jpeg?raw=true)

{% label %}
Figure 3: Wallets hold private keys to your public address on a blockchain

We can think of the box itself as a chunk of memory on the blockchain. The digital box location is referred to as a public key, or more intuitively as a public address, and can be shared with anyone in the same way you might share your address to receive mail at home. As the name implies, this is widely available to others on the network and it is the identifier others use to send digital assets to a box.

The key that can unlock the safety deposit box is referred to as the private key. Anyone with access to a box's private key has access to the box, which is why it's paramount to keep the key private - hence the name, private key.

# HD wallets

In this tutorial, we'll be building a type of wallet called a Hierarchical Deterministic (HD) wallet. We don't need to dive into the full definition of HD wallets here, but it's important to know that they enable the ability to store the private key as a 12-, 18-, or 24-word phrase referred to as a secret recovery phrase or mnemonic phrase. You'll be using a JavaScript library called [Bip39](https://github.com/bitcoinjs/bip39) to facilitate the generation of this phrase, which in turn can be converted into a private key to create a wallet.

{% sidenote title="Box 2.1: Why is the secret recovery phrase also called a mnemonic?" %}
The concept of a mnemonic phrase was introduced by Bitcoin Improvement Proposal 39 (BIP39). The term "mnemonic" refers to a pattern that makes something easier to remember. For instance, some of you might be familiar with the order of operations mnemonic in math: PEMDAS, or "please excuse my dear aunt Sally". The motivation for BIP39 was that "a mnemonic code or sentence is superior for human interaction compared to the handling of raw binary or hexadecimal representations of a wallet seed."

Run and open the app in your browser at [http://localhost:3000](http://localhost:3000). You'll notice the frontend offers the user the option to create a new wallet or import an existing wallet.

Clicking on **Create New Wallet** routes the user to a `/generate` page signaling to our user that by clicking **Generate**, the app will generate a key phrase and thus create a new wallet. But when we click **Generate**, we're routed to a `/phrase` page with an empty phrase container.

# Implementation 🧩

For the app to work, we need to implement a function that generates a phrase and uses it to create a wallet when the `/phrase` page renders. In your editor, navigate to `pages/phrase.tsx`.

In order to generate the phrase, we need to leverage an external library that satisfies the [BIP39 specification](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki), which set the standard for generating phrases for deterministic keys. 

Fortunately, there's the [Bip39 library](https://github.com/bitcoinjs/bip39). It provides us with the functionality we need to generate the phrase and later convert it into the seed we need to generate our Solana wallet keys.

We already installed the library when we ran `yarn` during set up because we included it into the pre-built app's [package.json](https://github.com/figment-networks/wallet-tutorial/blob/main/package.json). So all we need to do is import it:

```javascript
import * as Bip39 from "bip39";
```

## Recovery phrase

Recall that secret recovery phrases are also called mnemonics and Bip39 includes the method `generateMnemonic()` for generating mnemonic phrases. We can call it and replace the variable currently assigned to an empty string:

```javascript
const generatedMnemonic = Bip39.generateMnemonic();
```

This allows us to set the phrase in state and display it for our user to write down and store safely. With this step we have effectively created our wallet because the phrase by itself will allow its holder to access the account that matches it.

{% sidenote title="Box 2.2: The conservation of blockchain accounts" %}
Even though most of the community refers to accounts as being created, technically blockchain accounts can't be created or destroyed. These accounts are mathematical addresses in a system where their existence is established mathematically at inception by the system's architecture. It's more accurate to think of the mnemonic as obtaining the key to access a pre-existing, empty account.

## Keypairs and seeds

Before we can connect to the account on the blockchain, however, we need to convert the phrase into a form that the blockchain can understand. After all, mnemonic phrases are abstractions that translate a long, archaic number into a more human-friendly form.

We need to convert the phrase into bytes so the Solana web3.js library can use it to generate a `keypair` object. The `keypair` will be the wallet account consisting of a public key that can encrypt data and a private key that can decrypt data.

{% sidenote title="Box 2.3: Public-key cryptography" %}
At a high-level public-key cryptography is a cryptographic system that uses keypairs to transmit information securely. Each keypair consists of a public key, which is widely known across the system, and a private key, which should be kept secret by its owner. Suppose Bob wants to send a secure message to Alice while ensuring that, even if Eve intercepts it, she can't read it. Bob would encrypt the message using Alice's public key and then send the message. Because of something called a one-way function, the public key cannot be used to decrypt the message. The only way the message can be decrypted is with the private key that complements the public key used by Bob - that is, the private key that Alice holds. Bob can rest assured that no one but Alice will be able to read the message. The ability to encrypt information without the need to pre-share symmetric keys was a massive technological development. And the history behind these breakthroughs is well worth studying if you're so inclined. We've included a few suggestions as additional resources at the end of the tutorial.

By reviewing Solana's web3.js [documentation](https://solana-labs.github.io/solana-web3.js/index.html), we see that there's a `Keypair` class defined as "an account keypair used for signing transactions." This is exactly what we need to generate using the mnemonic phrase.

We notice that the `Keypair` class has a `fromSeed` method that generates a keypair from a 32-byte seed. It also mentions that the seed needs to be a `Uint8Array`, which means we'll need a way to convert our `string` phrase into a `Uint8Array`.

Going back to the Bip39 library we see a method called `mnemonicToSeedSync(mnemonic)` that returns some sort of `Buffer` object that looks like a list of hexadecimal numbers. We can test this by running it in our application and passing in the mnemonic we generated:

```javascript
const seed = Bip39.mnemonicToSeedSync(generatedMnemonic);
console.log(seed)

// console:
> Uint8Array(64);
```

It looks like we're close. The `Keypair` class requires a 32-byte `Uint8Array` and we're getting a 64-byte `Uint8Array`. We can slice the seed and keep only the first 32 bytes:

```javascript
const seed = Bip39.mnemonicToSeedSync(generatedMnemonic).slice(0, 32);
console.log(seed)

// console:
> Uint8Array(32);
```

With the seed in the right format, we can use `Keypair`'s `fromSeed` method to generate an account keypair:

```javascript
const newAccount = Keypair.fromSeed(seed);
```

We then set the account into context state and we have access to a Solana wallet.

Once you implement the functionality, make sure you save the recovery phrase before you click **Finish** on the browser. We'll be using it in [Step 6](https://learn.figment.io/tutorials/solana-wallet-step-6) to access the wallet once after logging out. 

Once you click on **Finish**, you'll be routed to the wallet page that displays the account's dashboard. You'll notice this page includes a few other features. We'll discuss the Network dropdown in [Step 3](https://learn.figment.io/tutorials/solana-wallet-step-3) while implementing the crucial functionality of showing users their balance.

In [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4) we'll dive into airdrops and make the **Airdrop** button functional, and in [Step 5](https://learn.figment.io/tutorials/solana-wallet-step-5) we'll enable the **Send** button and transfer funds.

##### _Listing 2.2: Code for creating a wallet_
```javascript
const generatedMnemonic = Bip39.generateMnemonic();
setMnemonic(generatedMnemonic);

const seed = Bip39.mnemonicToSeedSync(generatedMnemonic).slice(0, 32);
const newAccount = Keypair.fromSeed(seed);
setAccount(newAccount);
```

![](https://github.com/figment-networks/learn-tutorials/blob/master/solana/solana-wallet/assets/dashboard.png?raw=true)

# Challenge 🏋️ 

In your editor, navigate to `pages/phrase.tsx` and follow the steps included as comments to finish writing the function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 2.1](#listing-21-instructions-for-generating-a-phrase-and-creating-a-wallet) below.

##### _Listing 2.1: Instructions for generating a phrase and creating a wallet_
```javascript
useEffect(() => {
  // *Step 2*: implement a function that generates a mnemonic when the page renders, and uses it to create a wallet (i.e. account)
  // (a) review the import guidance on lines 9 and 11
  // (b) generate a mnemonic phrase by importing Bip39 and then implementing the appropriate method on the imported Bip39 instance
  // Documentation Reference: https://github.com/bitcoinjs/bip39
  const generatedMnemonic = "";

  // This line saves the mnemonic phrase to context state so we can display it for the wallet user to copy
  setMnemonic(generatedMnemonic);

  // (c) convert the mnemonic to seed bytes and make sure it's 32-bytes (Hint: console log the seed to see how many bytes you have vs how many you need)
  // Documentation Reference: https://github.com/bitcoinjs/bip39
  const seed = new Uint8Array();

  // (d) use the seed to generate a new account (i.e. a new keypair)
  // Documentation Reference:
  //   https://solana-labs.github.io/solana-web3.js/classes/Keypair.html
  //   https://solana-labs.github.io/solana-web3.js/classes/Keypair.html#fromSeed
  const newAccount = null;

  // This line sets the account to context state so it can be used by the app
  setAccount(newAccount);
}, []);
```

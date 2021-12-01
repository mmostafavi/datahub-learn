# Access

Getting access to accounts is obviously a critical feature for any wallet application, and given our choice of architecture, we don't have to worry about encrypting and decrypting passwords to log in and out.

The beauty of public-key cryptography is that the private key is effectively your password. And while securing crypto applications is more sophisticated than simply using your private key as a password, especially for industrial-strength wallets, we can achieve the basic functionality of accessing a wallet by leveraging what we learned in [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2) for creating wallets.

## Recovery phrase

Recall that we've built a type of wallet called a Hierarchical Deterministic (HD) wallet, which means we can map our 12-word phrase to a seed. Since the keypair that makes the account is derived from this seed, as long as we have the recovery phrase, we can get access to the keypair and therefore the account.

{% sidenote title="Box 6.1: Industrial-Strength Wallet Access" %}
In the context of an industrial-strength wallet, you would use the phrase as a recovery mechanism only. Day-to-day access would be enabled through the use of passwords. However, the analogy of the phrase serving as a type of very secure but somewhat cumbersome password still holds. Wallets like [MetaMask](https://metamask.io/) and [Phantom](https://phantom.app/) handle first-time access using recovery phrases and rely on locally encrypted password hashes for subsequent logins.

If you saved your test account's recovery phrase, we'll use it now. Feel free to click log out at the top right. Once we've finished this step, you'll once again have access to your devnet SOL.

# Implementation 🧩

Navigate to the app's home page at [http://localhost:3000](http://localhost:3000). If you click on the **Get Existing Wallet** button, you'll be routed to the page for importing the recovery phrase. But if you click on **Import** after pasting your phrase, you'll notice a message in the console that reads, "Recovery functionality not implemented yet!".

For this feature to work, we need to implement a function that uses the phrase to generate a seed, and then uses the seed to retrieve the keypair (i.e. access the account). 

Recall that in [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2) we first generated a mnemonic phrase with the Bip39 library by leveraging the `generateMnemonic` function. Then, in order to generate a keypair, we had to convert the phrase to a seed that could be be consumed by Solana's `Keypair` class to generate the `Keypair` that would grant us access to the account.

## Seed from phrase

In this case, we already have a phrase because the user imported it but the rest of the code mirrors [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2). We can convert the phrase into a seed and then leverage the `fromSeed` method from `Keypair` to access the account.

```javascript
const seed = Bip39.mnemonicToSeedSync(inputMnemonic).slice(0, 32);
const importedAccount = Keypair.fromSeed(seed);
```

With those two lines of code, we have effectively implemented authentication and allowed users to access their accounts from any device as long as they have their recovery phrase.

##### _Listing 6.2: Code for Recovering an Account_
```javascript
const handleImport = async (values: any) => {
  setLoading(true);
  const inputMnemonic = values.phrase.trim().toLowerCase();
  setMnemonic(inputMnemonic);

  const seed = Bip39.mnemonicToSeedSync(inputMnemonic).slice(0, 32);
  const importedAccount = Keypair.fromSeed(seed);
  setAccount(importedAccount);
};
```

This is a good point to reflect on what we've just completed. Clearly we built a crypto wallet capable of performing basic actions on Solana.

But taking a step back, we built something capable of performing actions that have long been the sole purview of giant financial institutions. Think about it. By understanding a few concepts, leveraging some libraries, and writing a few lines of code, you just built a trustless, decentralized bank. Let that sink in.

# Challenge 🏋️

Navigate to `pages/recover.tsx` in your editor and follow the steps included as comments to finish writing the function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 6.1](#listing-61-instructions-for-writing-recover-function) below.

##### _Listing 6.1: Instructions for writing recover function_
```javascript
// *Step 6*: implement a function that recovers an account based on a mnemonic phrase
const handleImport = async (values: any) => {
  console.log("Recovery functionality not implemented yet!");

  setLoading(true);
  const inputMnemonic = values.phrase.trim().toLowerCase();
  setMnemonic(inputMnemonic);

  // (a) review the import guidance on lines 9 and 11
  // (b) convert the mnemonic to seed bytes
  // Documentation Reference: https://github.com/bitcoinjs/bip39
  const seed = new Uint8Array();

  // (c) use the seed to import the account (i.e. keypair)
  // Documentation Reference:
  //   https://solana-labs.github.io/solana-web3.js/classes/Keypair.html
  //   https://solana-labs.github.io/solana-web3.js/classes/Keypair.html#fromSeed
  const importedAccount = null;
  setAccount(importedAccount);

  // (d) You can now delete the console.log statement since the function is implemented!
};
```

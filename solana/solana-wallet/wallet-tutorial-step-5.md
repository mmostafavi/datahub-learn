# Sign and send

Now that we've funded our account with test tokens on devnet, we can start to think about how we might use those funds. In a real-world application, we might want to swap SOL for another token, pay for goods or services in SOL, or even gift someone some SOL. To do that we need a way to let the network ledger know that our account should transfer funds to another account.

Thinking about it from first principles, we know we'll need a function that takes in two addresses - the sender and the recipient. We also know that we'll need to pass in an amount - the number of SOL or lamports that we want to send.

But there's one more important component: we need to prove to the network that we are in fact the owner of those funds and that we approve the transfer.

## Paper checks

Consider a traditional paper check that you might use to pay your landlord. The check has your name and address printed on the top left. It includes a field for you to write the recipient's name along with a field for you to write the amount you're paying. Finally, it includes a field for you to sign the check to validate to the bank that you're approving the transfer.

![Figure 8: Crypto transactions are like digital checks (with real signatures)](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/check.jpeg)

{% label %}
Figure 8: Crypto transactions are like digital checks (with real signatures)

Let's pretend that banks actually use those signatures to validate that you actually signed the check and it's not someone else forging your signature (spoiler alert: they don't for the most part).

## Blockchain transfers

The blockchain model of transferring funds is pretty similar. You need a way to sign the transaction so the network can confirm it as valid and change the corresponding balances.

The concept of cryptographic signing is fascinating, but well beyond the scope of this tutorial. We'll provide [additional resources](#additional-resources) at the end if you want to explore it more, but recall that your keys, along with hashing algorithms and a one-way function, are designed for this very purpose. The signature authenticates the sender and proves to the recipient that the message hasn't been compromised.

With those building blocks in mind, we're ready to search the docs for a way to send and confirm a transaction.

# Implementation 🧩

If you click on the **Send** button on the wallet dashboard, a drawer component opens up that shows a form structured as a paper check thus building on our analogy above. It includes all the components of a transaction but the **Sign and Send** button doesn't work yet.

Navigating to `components/TransactionLayout/index.tsx` we see a partially implemented `transfer` function.

Based on [Step 3](https://learn.figment.io/tutorials/solana-wallet-step-3) and [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4) we know that we're going to need a connection to the network. We can once again leverage our previous code:

```javascript
const connection = new Connection(clusterApiUrl(network), "confirmed");
```

## Transactions

We can guess that we'll need to build some sort of transaction object and send it through the connection, but it's not immediately obvious how we might do that. By searching the docs, it looks like there are two ways to send transactions - the `sendTransaction` method in `Connection` and the general function `sendAndConfirmTransaction`.

We're going to want to request a confirmation so we can update our balance after sending the transaction, so the latter seems like a good option to try.

Reading through the function's specs, it looks like we need to pass in a `connection`, some sort of `transaction` object and a `signers array`. Moreover, the description says that this function will "sign, send and confirm a transaction" so it looks like we're on the right track. We can begin by importing this function and using it, knowing that we're going to build out its inputs next.

```javascript
const confirmation = await sendAndConfirmTransaction(
  connection,
  transaction,
  signers
);
```

We already have a `connection`, so we can immediately turn to the `transaction` parameter. Following the link to the `Transaction` class in the docs, it looks like to create a `transaction` object, we can use its constructor.

```javascript
const transaction = new Transaction()
console.log(transaction)

// console:
> Transaction {signatures: Array(0), feePayer: undefined, instructions: Array(0), recentBlockhash: undefined, nonceInfo: undefined}
```

Looking at the transaction we created, it clearly lacks any of the components we need for a valid transaction like `sender`, `recipient` and `amount`, but it seems to have a familiar structure that we should be able to populate. You could research each of them, but let's jump right into the `instructions` property as it seems like a promising place to start.

Unfortunately, the docs aren't very intuitive in how to progress but we can leverage our technical sophistication. There's a very useful class called `SystemProgram` with a `transfer` method, which "generates a transaction instruction that transfers lamports from one account to another." This seems like exactly what we need.

## Transfer

The `transfer` method takes in a `TransferParams` object that requires a `sender`, `recipient` and `amount` in lamports. That matches the data we're hoping to use for our transaction and we can get that from the account in context state and the form inputs. The resulting instructions look like this:

```javascript
const instructions = SystemProgram.transfer({
  fromPubkey: account.publicKey,
  toPubkey: new PublicKey(form.to),
  lamports: form.amount,
});
```

Notice that we have to instantiate a `PublicKey` for the recipient because the form is passing in the key as a string but the `toPubkey` property expects a `PublicKey` type. To incorporate these into our transaction, we can use the `add` method:

```javascript
transaction.add(instructions);
```

Or with a bit of on the fly refactoring, we can simply instantiate the transaction after creating the instructions and add them immediately:

```javascript
const transaction = new Transaction().add(instructions);
```

We have two out of the three parameters ready for the `sendAndConfirmTransaction` function - `connection` and `transaction`.

## Signing

Now we need the `signers array`. From the function's specification, we know `signers` will be an array with at least one `Signer` object. Reviewing the `Signer` type in the docs, it looks like it's an object with two properties - `publicKey` and `privateKey`. We can get both from the `account`, so we can build the `signers array`.

```javascript
const signers = [
  {
    publicKey: account.publicKey,
    secretKey: account.secretKey,
  },
];
```

## Send and confirm

Now that all three parameters are complete, we can finally call `sendAndConfirmTransaction` and await its confirmation.

```javascript
const confirmation = await sendAndConfirmTransaction(
  connection,
  transaction,
  signers
);
```

With that we have a fully functional feature capable of transferring funds between Solana accounts. To complete the feature, we need to make sure we call the `refreshBalance` function to update the account's balance after transferring funds.

```javascript
const updatedBalance = await refreshBalance(network, account);
setBalance(updatedBalance);
```

## Test driving

If you haven't already, airdrop some devnet SOL into your account and test transferring funds to another account. If you have another account to use, feel free to do that. Otherwise you can send the devnet SOL back to the smart contract that sent them to you (these are called faucets)! Here's its public address: 9B5XszUGdMaxCZ7uSQhPzdks5ZQSmWxrmzCSvtJ6Ns6g.

Once you fill in the public address of your recipient and the amount, say one million lamports, the **Sign and Send** button will be enabled. Once you click **Sign and Send** you will see a successful message displayed at the top of the page along with a link to the [Solana Block Explorer](https://explorer.solana.com/?cluster=devnet) at the top left of the check.

## Block explorer

The [Solana Block Explorer](https://explorer.solana.com/?cluster=devnet) is a simple dashboard that allows you to search for specific blocks, accounts, transactions, contracts and tokens by network. It displays all the information related to the item you searched for.

![Figure 9: Oh the places you'll go!](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/explore.jpeg)

{% label %}
Figure 9: Oh the places you'll go!

In this case, if you click the link on the check, you'll be able to see a basic overview of the transfer you just issued. In the middle of the page, you'll see our transfer's information - mainly, the sender (your public address) and how much SOL you sent; and the recipient (the other public address) and how much SOL they received. In fact, you can check whether you sent the SOL back to the same contract that sent it to you by clicking on the previous transaction in the Transaction History.

You might notice a field labeled "Fee (SOL)". If you scroll to the Account Input(s), you'll also notice that it was charged to your account. While the recipient received the funds you specified, your account was deducted the amount you sent plus a small fee. These [transaction fees](https://docs.solana.com/transaction_fees) are designed to reward validators for the compute power spent to process transactions.

At this point, your Solana wallet is almost complete, except for one major flaw. You can create a wallet, and even transfer funds from it. But you can't access an existing wallet. We'll fix that in [Step 6](https://learn.figment.io/tutorials/solana-wallet-step-6) where we'll once again leverage the Bip39 library to access an account based on a mnemonic phrase.

##### _Listing 5.2: Code for transferring funds_
```javascript
const transfer = async () => {
  if (!account) return;

  try {
    setTransactionSig("");

    const connection = new Connection(clusterApiUrl(network), "confirmed");

    const instructions = SystemProgram.transfer({
      fromPubkey: account.publicKey,
      toPubkey: new PublicKey(form.to),
      lamports: form.amount,
    });

    const transaction = new Transaction().add(instructions);

    const signers = [
      {
        publicKey: account.publicKey,
        secretKey: account.secretKey,
      },
    ];

    setSending(true);
    const confirmation = await sendAndConfirmTransaction(
      connection,
      transaction,
      signers
    );
    setTransactionSig(confirmation);
    setSending(false);

    const updatedBalance = await refreshBalance(network, account);
    setBalance(updatedBalance);
    message.success(`Transaction confirmed`);
  } catch (error) {
    message.error("Transaction failed, please check your inputs and try again");
    console.log(error);
  }
};
```

# Challenge 🏋️

Navigate to `components/TransactionLayout/index.tsx` in your editor and follow the steps included as comments to finish writing the `transfer` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 5.1](#listing-51-instructions-for-writing-transfer-function) below.

##### _Listing 5.1: Instructions for writing transfer function_
```javascript
// *Step 5*: implement a function that transfer funds
const transfer = async () => {
  // This line ensures the function returns before running if no account has been set
  if (!account) return;

  try {
    // (a) review the import guidance on line 1
    // (b) instantiate a connection using clusterApiUrl with the active network passed in as an argument
    // Documentation References:
    //   https://solana-labs.github.io/solana-web3.js/classes/Connection.html
    //   https://solana-labs.github.io/solana-web3.js/modules.html#clusterApiUrl
    console.log("Sign and Send not yet implemented!");
    const connection = "";
    setTransactionSig("");
    // (c) leverage the SystemProgram class to create transfer instructions that include your account's public key, the public key from your sender field in the form, and the amount from the form
    // Documentation Reference:
    //   https://solana-labs.github.io/solana-web3.js/classes/SystemProgram.html
    //   https://solana-labs.github.io/solana-web3.js/classes/SystemProgram.html#transfer
    const instructions = {};

    // (d) instantiate a transaction object and add the instructions
    // Documentation Reference:
    //   https://solana-labs.github.io/solana-web3.js/classes/Transaction.html
    //   https://solana-labs.github.io/solana-web3.js/classes/Transaction.html#add
    const transaction = {};

    // (e) use your account to create a signers interface
    // Documentation Reference:
    //   https://solana-labs.github.io/solana-web3.js/interfaces/Signer.html
    //   note: signers is an array with a single item - an object with two properties
    const signers = [{}];

    setSending(true);
    // (f) send the transaction and await its confirmation
    // Documentation Reference: https://solana-labs.github.io/solana-web3.js/modules.html#sendAndConfirmTransaction
    const confirmation = "";
    setTransactionSig(confirmation);
    setSending(false);

    const updatedBalance = await refreshBalance(network, account);
    setBalance(updatedBalance);
    message.success(`Transaction confirmed`);
    // (g) You can now delete the console.log statement since the function is implemented!
  } catch (error) {
    console.log(error);
    message.error("Transaction failed, please check your inputs and try again");
  }
};
```

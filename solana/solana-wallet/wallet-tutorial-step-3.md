# Blockchain requests

Crypto wallets serve one primary function - storing your private keys. By storing your private keys, they allow you to manage - transfer, receive, organize - your digital assets. Part of that functionality requires the wallet to retrieve certain data that is stored onchain to display on your user dashboard.

A balance represents a certain amount of cryptocurrency or tokens held by an account. If you think of the blockchain as a database that keeps ownership records, and of the public keys as the owner IDs, then you can think of the balances as an integer column in the database that tracks how much of a certain token each owner holds.

We'll be connecting to one of Solana's networks, and fetching the balance for the account we just created. Later in [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4), we'll be funding the account with test tokens using the airdrop function.

## Networks

The concept of various networks for a single protocol is similar to that of different environments for an app (e.g. development, test, production, etc). Typically blockchain protocols have a main network or mainnet, which refers to the production blockchain with real economic value and official transactions, and at least one experimentation network, which refers to an identical blockchain used to test features before they go live on mainnet.

![Figure 4: It's always important to test on devnet before deploying on mainnet](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/solana/solana-wallet/assets/consultant.jpeg)

{% label %}
Figure 4: It's always important to test on devnet before deploying on mainnet

Solana has a production network called mainnet and two experimentation networks called testnet and devnet. Solana's devnet is designed for developers and users to play with various features and debug dApps before launching on mainnet with real economic consequences. The testnet is where Solana tests potential protocol updates.

You'll notice the wallet includes a dropdown at the top-right that allows users to select what network they want to connect to. This allows the wallet to manage assets specific to the connected network. Our default network is devnet since we'll be using it to receive test tokens in [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4) and then transfer them in [Step 5](https://learn.figment.io/tutorials/solana-wallet-step-5), but the functionality we'll build will work for any of the Solana networks.

# Implementation 🧩

In [Step 2](https://learn.figment.io/tutorials/solana-wallet-step-2), we discussed how a wallet is more like a keychain that holds keypairs representing an account address and the key to access it. We built a function that allowed us to generate a unique account and the corresponding phrase that works like a password for accessing the account. Now we need to connect with the Solana blockchain so we can fetch the account's balance, which at this point should be zero because we just created it.

## Connecting

The first step for interacting with any blockchain is to instantiate a connection. Conveniently there's a `Connection` class in web3.js designed to do just that. By reviewing the documentation, we notice the `Connection` constructor requires two arguments: `endpoint: string` and `commitmentOrConfig?: Commitment | ConnectionConfig`.

The description for `endpoint` mentions that it's a "URL to the fullnode JSON RPC endpoint". Since we don't have a URL for the connection, we need to either find one from Solana or look for a function that will return a URL. By searching the web3.js docs for "URL", we see that there's a function called `clusterApiUrl` that returns the "RPC API URL for the specified cluster". Moreover, if we review the `Cluster` type, we see that it refers to the network we want to connect to.

As for `commitmentOrConfig`, it looks like the definition for `Commitment` types describes it as the "level of commitment desired when querying state", which is a meaningless definition to us at this point. However, it looks like `Commitment` can be one of several strings so we should choose one and test the function. In this case, we can choose "confirmed" as a reasonable guess and move forward.

{% sidenote title="Box 3.1: A Note on Technical Sophistication" %}
In his ground-breaking series on software development, Michael Hartl and the [Learn Enough](https://www.learnenough.com/) team introduced the concept of _technical sophistication_ as a key attribute of the technical development process. They define it as "the combination of hard and soft skills that make it seem like you can magically solve any technical problem." Figuring things out and being resourceful is at least half the battle, and sometimes the whole war. When in doubt, refer to the [xkcd flowchart](https://m.xkcd.com/627/).

Putting the above together, we can start implementing the `refreshBalance` function with a connection instance in `utils/index.ts`:

```javascript
const connection = new Connection(clusterApiUrl(network), "confirmed");
console.log(connection)

// console:
> Connection {_commitment: 'confirmed', _confirmTransactionInitialTimeout: undefined, _rpcEndpoint: 'https://api.devnet.solana.com', _rpcWsEndpoint: 'wss://api.devnet.solana.com/', _rpcClient: ClientBrowser, …}
```

## Fetching data

Now that we have a connection, we need to fetch our account's balance. We might speculate that there should be a `getBalance` function that takes an account's public address as a parameter, and returns the account's balance. Searching for the keyword "balance" in the web3.js docs, we can see that, not only is there a `getBalance` method, but it's a `Connection` class method.

Reviewing the `getBalance` method on `Connection`, we can see it expects the account's public key as a parameter.

```javascript
const publicKey = account.publicKey;
const balance = await connection.getBalance(publicKey);
console.log(balance);

// console:
> 0;
```

There are a few ways to structure this part of the function. We've chosen to assign the `publicKey` variable to the account's public key, and then pass that into `getBalance` to query the network for the balance.

From the docs, we know `getBalance` returns a promise so we use `await` and assign its return value to the `balance` variable. By logging `balance`, we can see our account has zero balance as expected.

But that's a bit anticlimactic. We know it's a new account and it should have zero balance. Nothing changed on the frontend because the default value was zero. We need to fund the account to see the balance change, and we'll do just that in [Step 4](https://learn.figment.io/tutorials/solana-wallet-step-4).

##### _Listing 3.2: Code for fetching a balance_
```javascript
const refreshBalance = async (network: Cluster, account: Keypair | null) => {
  if (!account) return;

  try {
    const connection = new Connection(clusterApiUrl(network), "confirmed");
    const publicKey = account.publicKey;
    const balance = await connection.getBalance(publicKey);
    return balance;
  } catch (error) {
    console.log(error);
    return 0;
  }
};
```

# Challenge 🏋️

If you open the browser's console from the `/wallet` page, you'll notice a message that reads, "Balance functionality not implemented yet!". Navigate to `utils/index.ts` in your editor and follow the steps included as comments to finish writing the `refreshBalance` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 3.1](#listing-31-instructions-for-fetching-an-accounts-balance) below.

##### _Listing 3.1: Instructions for fetching an account's balance_
```javascript
// *Step 3*: implement a function that gets an account's balance
const refreshBalance = async (network: Cluster, account: Keypair | null) => {
  // This line ensures the function returns before running if no account has been set
  if (!account) return 0;

  try {
    // (a) review the import guidance on line 1
    // (b) instantiate a connection using clusterApiUrl with the active network passed in as an argument
    // Documentation References:
    //   https://solana-labs.github.io/solana-web3.js/classes/Connection.html
    //   https://solana-labs.github.io/solana-web3.js/modules.html#clusterApiUrl
    console.log("Balance functionality not implemented yet!");
    const connection = "";

    // (c) get the key using one of the accessors on the account passed in as an argument
    // Documentation Reference: https://solana-labs.github.io/solana-web3.js/classes/Keypair.html
    const publicKey = "";

    // (d) get the account's balance using the connection instance
    // Documentation Reference: https://solana-labs.github.io/solana-web3.js/classes/Connection.html
    const balance = 0;

    return balance;
    // (e) You can now delete the console.log statement since the function is implemented!
  } catch (error) {
    console.log(error);
    return 0;
  }
};
```

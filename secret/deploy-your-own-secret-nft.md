# Introduction

For a high level introduction to Non-Fungible-Tokens see the [first installment of this series](https://learn.figment.io/tutorials/create-your-first-secret-nft). In this tutorial we will download and compile the [snip721 reference implementation](https://github.com/baedrik/snip721-reference-impl), deploy it onto the secret testnet and interact with the contract, minting your own secret NFTs. Unlike the [previous tutorial](https://learn.figment.io/tutorials/create-your-first-secret-nft) we will configure the contract ourselves and learn about access right management of secret contracts and tokens on the way.

{% embed url="https://youtu.be/jRuSOos9ig4" %}

# Prerequisites

This tutorial assumes that you have completed the [Secret Pathway](https://learn.figment.io/protocols/secret) already, as we will be building upon that foundation of knowledge and skill. If you have not already done so, you would be wise to take the time to complete the Pathway. We will start with the same project folder as in section 5 of the Pathway.

# Requirements

* The latest version of [NodeJS](https://nodejs.org/en/) installed \(use of nvm, the node version manager, is _encouraged_ for web3 developers\)
* A code editor like [VSCode](https://code.visualstudio.com/Download), Theia, Atom, _etc_.
* Required JavaScript packages –
  * [secretjs](https://www.npmjs.com/package/secretjs) - for the Secret Network JavaScript API
  * [dotenv](https://www.npmjs.com/package/dotenv) - for working with environment variables
* [Rust](https://rustup.rs) + [docker](https://docs.docker.com/get-docker/) toolchain to compile secret contracts

## Generate the contract

```text
cargo generate --git https://github.com/baedrik/snip721-reference-impl --name my-snip721
```

This git project is a reference implementation for tokens based on the [snip721 standard](https://github.com/SecretFoundation/SNIPs/blob/master/SNIP-721.md), which creates tokens that are loosely based on the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) specification and are a superset of [CW-721](https://github.com/CosmWasm/cosmwasm-plus/blob/master/packages/cw721/README.md), making them compatible with both the Ethereum and Cosmos tokens.

You can have a look at the generated files by stepping into the folder using:

```text
cd my-snip721 && ls
```

Most of the files in here should look very familiar from what you saw in part 5 of the Secret pathway, with the main difference that the `src` folder contains more files, due to the complexity of the contract

```text
Cargo.lock  Developing.md  LICENSE  Publishing.md  examples      schema  tests
Cargo.toml  Importing.md   NOTICE   README.md      rustfmt.toml  src
```

While we will keep most of the files as-is during this tutorial, we will alter some of them for the tutorial to deepen your understanding what is happening behind the scenes. For now the goal is to compile the contract and upload it to the Secret testnet.

## Compile the contract

To compile the smart contract into a WebAssembly \(.wasm\) binary, run this command in the terminal:

```text
cargo wasm
```

Before deploying or storing the contract on the testnet, you need to run the [secret contract optimizer](https://hub.docker.com/r/enigmampc/secret-contract-optimizer).

## Optimize compiled wasm

```text
docker run --rm -v "$(pwd)":/contract \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  enigmampc/secret-contract-optimizer
```

This will output an optimized build file, `contract.wasm.gz`, ready to be stored on the Secret Network. _Note that Windows users should replace the backslashes `\` in the command with carets `^` or optionally remove them and paste the command as a single line._

While `secretcli` supports uploading the compressed file, we'll be using SecretJS which expects the uncompressed WASM file, so lets unpack the optimized `contract.wasm` with this terminal command:

```text
gunzip contract.wasm.gz
```

You should be able to find a newly created file `contract.wasm` in the current directory.

For uploading the compiled contract we will reuse the knowledge you gained during part 5 of the secret pathway.

## Uploading the contract

We are now switching back to the project root directory and will start writing some Javascript modules to manage our newly created contract. For this we will need the `secretjs` and `dotenv` packages. If you haven't set it up by now I recommend a quick detour to the first step of the secret pathway to do so:

{% page-ref page="intro-pathway-secret-basics/1.-connecting-to-a-secret-node-using-datahub.md" %}

After setting everything up we start by creating a new file `deploy-nft.js` in the root project directory and add the code below:

```javascript
const {
  EnigmaUtils,
  Secp256k1Pen,
  SigningCosmWasmClient,
  pubkeyToAddress,
  encodeSecp256k1Pubkey,
} = require("secretjs");
const fs = require("fs");

// Load environment variables
require("dotenv").config();

const customFees = {
  upload: {
    amount: [{ amount: "4000000", denom: "uscrt" }],
    gas: "4000000",
  },
  init: {
    amount: [{ amount: "500000", denom: "uscrt" }],
    gas: "500000",
  },
  exec: {
    amount: [{ amount: "500000", denom: "uscrt" }],
    gas: "500000",
  },
  send: {
    amount: [{ amount: "80000", denom: "uscrt" }],
    gas: "80000",
  },
};

const main = async () => {
  const httpUrl = process.env.SECRET_REST_URL;

  // Use key created in tutorial #2
  const mnemonic = process.env.MNEMONIC;

  // A pen is the most basic tool you can think of for signing.
  // This wraps a single keypair and allows for signing.
  const signingPen = await Secp256k1Pen.fromMnemonic(mnemonic).catch((err) => {
    throw new Error(`Could not get signing pen: ${err}`);
  });

  // Get the public key
  const pubkey = encodeSecp256k1Pubkey(signingPen.pubkey);

  // get the wallet address
  const accAddress = pubkeyToAddress(pubkey, "secret");

  // 1. Initialize client

  // 2. Upload the contract wasm

  // 3. Create an instance of the NFT contract init msg

  const initMsg = {};
  const contract = await client
    .instantiate(
      codeId,
      initMsg,
      `My Snip721${Math.ceil(Math.random() * 10000)}`
    )
    .catch((err) => {
      throw new Error(`Could not instantiate contract: ${err}`);
    });
  const { contractAddress } = contract;
  console.log("contract: ", contract, "address:", contractAddress);
};

main().catch((err) => {
  console.error(err);
});
```

## Initialize the client

In the `deploy-ft.js` file, under the comment `// 1. Initialize client` add the following code :

```javascript
  const txEncryptionSeed = EnigmaUtils.GenerateNewSeed();

  const client = new SigningCosmWasmClient(
    httpUrl,
    accAddress,
    (signBytes) => signingPen.sign(signBytes),
    txEncryptionSeed, customFees,
  );
  console.log(`Wallet address=${accAddress}`);
```

## Upload the contract wasm

Under the comment `// 2. Upload the contract wasm` add the following code :

```javascript
  const wasm = fs.readFileSync('my-snip721/contract.wasm');
  console.log('Uploading contract');
  const uploadReceipt = await client.upload(wasm, {})
    .catch((err) => { throw new Error(`Could not upload contract: ${err}`); });

  // Get the code ID from the receipt
  const { codeId } = uploadReceipt;
```

Ensure that if you have changed the name of the contract folder, you also change it accordingly when passing it to `readFileSync()`:

```javascript
  const wasm = fs.readFileSync('my-snip721/contract.wasm');
```

# Instantiating the contract

Similar to what you have seen before, we first got the `codeId` from the upload receipt and then defined the `initMsg` to instantiate the contract. In this case the initMsg is more complex that for a simple counter and allows us to configure the secret NFT to our liking.

Open up the `msg.rs` file within the `src` folder of the contract code. You will see two structs: `InitMsg` and `InitConfig` in there, which describe the settings object we can pass to our contract on initialization. For most values a sensible default is predefined, giving the most privacy-preserving behavior.

Lets have a look at the different fields and what part of the contract they control:

## InitMsg

| Name | Type | Description | Optional | Value If Omitted |
| :--- | :--- | :--- | :--- | :--- |
| name | string | Name of the token contract | no |  |
| symbol | string | Token contract symbol | no |  |
| admin | string \(HumanAddr\) | Address to be given admin authority | yes | env.message.sender |
| entropy | string | String used as entropy when generating random viewing keys | no |  |
| config | [Config \(see below\)](deploy-your-own-secret-nft.md#config) | Privacy configuration for the contract | yes | defined below |
| post\_init\_callback | [PostInitCallback \(see below\)](deploy-your-own-secret-nft.md#postinitcallback) | Information used to perform a callback message after initialization | yes | nothing |

## InitConfig

| Name | Type | Description | Optional | Value If Omitted |
| :--- | :--- | :--- | :--- | :--- |
| public\_token\_supply | bool | This config value indicates whether the token IDs and the number of tokens controlled by the contract are public.  If the token supply is private, only minters can view the token IDs and number of tokens controlled by the contract | yes | false |
| public\_owner | bool | This config value indicates whether token ownership is public or private by default.  Regardless of this setting a user has the ability to change whether the ownership of their tokens is public or private | yes | false |
| enable\_sealed\_metadata | bool | This config value indicates whether sealed metadata should be enabled.  If sealed metadata is enabled, the private metadata of a newly minted token is not viewable by anyone, not even the owner, until the owner calls the Reveal message.  When Reveal is called, the sealed metadata is irreversibly unwrapped and moved to the public metadata \(as default\).  If `unwrapped_metadata_is_private` is set to true, the sealed metadata will remain as private metadata after unwrapping, but the owner \(and anyone the owner has whitelisted\) will now be able to see it.  Anyone will be able to query the token to know whether it has been unwrapped.  This simulates buying/selling a wrapped card that no one knows which card it is until it is unwrapped. If sealed metadata is not enabled, all tokens are considered unwrapped when minted | yes | false |
| unwrapped\_metadata\_is\_private | bool | This config value indicates if the Reveal message should keep the sealed metadata private after unwrapping.  This config value is ignored if sealed metadata is not enabled | yes | false |
| minter\_may\_update\_metadata | bool | This config value indicates whether a minter is permitted to update a token's metadata | yes | true |
| owner\_may\_update\_metadata | bool | This config value indicates whether the owner of a token is permitted to update a token's metadata | yes | false |
| enable\_burn | bool | This config value indicates whether burn functionality is enabled | yes | false |

For this tutorial we will keep most of the default values and just change the name, symbol and the default ownership visibility of the token. Insert this piece of code below `// 3. Create an instance of the NFT contract init msg` and add some values for name and symbol, as well as some random string for the entropy. Remember to keep the strings enclosed within the single quotes.

```javascript
  /// name of token contract
  name: '',
  /// token contract symbol
  symbol: '',
  /// entropy used for prng seed
  entropy: '',
  /// optional privacy configuration for the contract
  config: {
      public_owner: true
  },
```

Let's run the code:

```text
node deploy-nft.js
```

If it went well, you should see similar output:

```text
Uploading contract
contract:  {
  contractAddress: 'secret1g0t7sggeh89k27xa2vux5rnpc3ly4a9c0u8724',
  logs: [ { msg_index: 0, log: '', events: [Array] } ],
  transactionHash: 'F5E734014EA3108B071B3EA390E58FC41FA0DB28D1F49FE7A652C53E482AA0D9',
  data: '43D7E82119B9CB6578DD53386A0E61C47E4AF4B8'
} address: secret1g0t7sggeh89k27xa2vux5rnpc3ly4a9c0u8724
```

**Unable to deploy your contract or initializing it using deploy-ft.js**

Let's check for some common causes:

* First, make sure you have `.env` file saved and it's in the correct format as given in the tutorial.
* If you're getting an error message like `UnauthorizedError: { "message":"Invalid authentication credentials"` then make sure to replace the &lt;API\_KEY&gt; with your correct API key which you copied from your DataHub Dashboard.
* If you are getting `Error: Cannot find module 'secretjs'` make sure you installed the packages correctly using `npm install --save secretjs dotenv @iov/crypto`
* If you see `Error: ENOENT: no such file or directory, open 'my-snip721/contract.wasm'` make sure that file path you are using in your deploy script points to the generated `contract.wasm`
* If still, you're experiencing the same issue, for help reach out to us on [Discord](https://discord.gg/fszyM7K) or [Forum](https://community.figemnt.io)


After this executed successfully you can take the program you created in the [first tutorial](https://learn.figment.io/tutorials/create-your-first-secret-nft), change the contract address to the one of your contract and interact with it in the same way!

# Conclusion

Congratulations! We have made it to the end of the first installment of this Secret NFT series. We have covered a lot of information, and I feel you can really be proud of what you have achieved. Just to recap:

* You created and compiled your very own secretNFT contract, based on snip721
* You explored different parameters to configure the contract to your liking
* You created an instance of your contract on the secret testnet, ready to be interacted with

This is a solid foundation to play with and build upon!

# About the author

This tutorial was created by [Florian Uhde](https://twitter.com/florianuhde), a software engineer and game developer with a passion for blockchain, creativity and systemic design. You can get in touch with the author on [Figment Forum](https://community.figment.io/u/floar) if you have any queries pertaining to the tutorial, secretNFTs, etc.

# References

snip721 Reference Implementation: [Github Repo](https://github.com/baedrik/snip721-reference-impl)

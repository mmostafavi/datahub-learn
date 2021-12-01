# Introduction

In this tutorial, we will be creating your first Terra account using the Terra SDK and [**DataHub**](https://figment.io/datahub-waitlist/). 

# Prerequisites

Please make sure that you completed tutorial \#1 of the Terra Pathway Tutorial, "Connecting to a Terra node with DataHub", before beginning. We will be building on top of the Nodejs application you created in the previous tutorial.

## Creating Account

Time to start building! We are finally ready to jump in and create our Terra account on the Tequila testnet.

Create a file `create_account.js` with the snippet below:

```javascript
const { LCDClient, MnemonicKey } = require('@terra-money/terra.js');
require('dotenv').config();

const main = async () => {
  const terra = new LCDClient({
    URL: process.env.TERRA_NODE_URL,
    chainID: process.env.TERRA_CHAIN_ID,
  });

  const mk = new MnemonicKey();

  console.log('address: ', mk.accAddress);
  console.log('mnemonic: ', mk.mnemonic);
}

main().then(resp => {
  console.log(resp);
}).catch(err => {
  console.log(err)
})
```

Here is a walkthrough of for the code above:

* Import the Terra SDK and initialize dotenv which will load variables from `.env` file to become available from `process.env.*`
* Create an `LCDClient` using the URL and `chainID` from the `.env` file
* Create a randomly generated mnemonic key which is essentially an account address and a list of secret words associated with this account
* Output mnemonic and `accAddress`. This is important because we will need to store those values in our `.env` file

Once this file is in place, you can run the `create_account.js` script:

```javascript
node create_account.js
```

If everything works, you should see your address and mnemonic in your console. Please make sure that you always backup your mnemonic as this is very sensitive data that allows you to restore your account.

{% hint style="info" %}
Never share your mnemonic with anyone. Never commit it to a repo. **Remember! You can share your account address, but never your mnemonic.**
{% endhint %}

In order to make sure our code doesn't leak sensitive data, we can use our `.env` file where we can store both our account address and our mnemonic. Open up `.env` and add to it below the following lines:

```javascript
...
ADDRESS='<YOUR ADDRESS>'
MNEMONIC='<YOUR MNEMONIC>'
```

Great! Now both `ADDRESS` and `MNEMONIC` will be available for use in our Nodejs scripts.

## Getting testnet tokens

Congratulations! You’ve just created your Tequila account.

You can go to this URL [https://tequila.stake.id/?\#/](https://tequila.stake.id/?#/) to check your address with the search bar. Once you enter your address, you should see something similar to:

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/terra-1.png)

Now that you have an account, how about getting some tokens? Let’s get you set up! 

Copy your address to the clipboard and head to [Terra Faucet](https://faucet.terra.money/). Check “I’m not a robot”, paste in your address, select "Luna", and click `Send me tokens`.

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/terra-2.png)

After a few seconds you should see a message pop up on the top right corner of the screen:

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/terra-3.png)

That's it! Now head back to the Stake ID app, refresh the page and enjoy the view of 1000 LUNAs in your account.

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/terra-4.png)

Great work. You’ve earned it!

# Conclusion

You now have an account on the Terra Tequila testnet and your secret information stored in the `.env` file. You also made sure that your account balance is loaded with testnet LUNAs.

The complete code for this tutorial can be found [**here**](https://github.com/figment-networks/tutorials/blob/main/terra/2_creating_account/create_account.js). 

# Next Steps

In the next tutorial, we will be looking at different queries we can run against a Terra node to get the information such as:

* blockchain details
* account details
* Luna price
* proposals

If you had any difficulties following this tutorial or simply want to discuss Terra tech with us you can [join our community today](https://discord.gg/fszyM7K)!


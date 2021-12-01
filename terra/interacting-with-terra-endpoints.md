# Introduction

In order to benefit from the full potential of the Terra blockchain, you will first need to connect to a Terra node. 

In this tutorial, we will use the Terra SDK to connect to a Terra node hosted by ****[**DataHub**](https://figment.io/datahub-waitlist/). We will be building a simple Nodejs application, which we will develop further during the Terra Pathway. 

More details about Terra.js can be found [**here**](https://terra-project.github.io/terra.js/).

## **Creating the NodeJS App**

Before we can jump in and start using the Terra SDK, we need to set up our project and all required dependencies. 

{% hint style="info" %}
Please make sure that you have Node 10+ installed on your machine.
{% endhint %}

The next step is to create a directory for your project where you will initialize the new Nodejs project. You can do this using the following command:

```javascript
npm init
```

You will then be prompted with several questions. For the purpose of this tutorial, you can safely press `Enter` for every one of these questions \(which will create `package.json` with the default options\). This should create a `package.json` file in the root of your directory that looks similar to:

```javascript
{
  "name": "terra",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
   }
}
```

## **Installing packages**

Now that we have our Nodejs application set up we can install the required packages:  


* **dotenv** - for keeping sensitive data
* **terra.js** - Terra SDK

```javascript
npm install --save dotenv @terra-money/terra.js
```

## **Store sensitive data**

When we have that out of our way, we can continue by creating an `.env` file which will hold all our sensitive data.

You should never commit this file to a git repository so make sure to create a `.gitignore` file and the following command:

```javascript
.env
```

Now open `.env` file and add the first environmental variables:

* **TERRA\_NODE\_URL** - url to DH Tequila node
* **TERRA\_CHAIN\_ID** - chain id which should be set to `tequila-0004`

Your `.env` file should look similar to:

```javascript
TERRA_NODE_URL='https://tequila-0004--lcd--archive.datahub.figment.io/apikey/<YOUR API KEY>/'
TERRA_CHAIN_ID='tequila-0004'
```

{% hint style="info" %}
**Warning! Make sure that you don’t forget about the trailing slash for the TERRA\_NODE\_URL**
{% endhint %}

## **Conclusion**

Now that we have successfully connected to a Terra node using [**DataHub**](https://figment.io/datahub-waitlist/), we are ready to move on to the next tutorial.

The complete code for this tutorial can be found [**here**](https://github.com/figment-networks/tutorials/blob/main/terra/1_connecting_to_node/connect.js).   


## **Next steps**

In the next tutorial, we will be creating your very first Terra account using the Terra SDK.

If you had any difficulties following this tutorial or simply want to discuss Terra tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!                        ****


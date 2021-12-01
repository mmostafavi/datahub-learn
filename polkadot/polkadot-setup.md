# 🧩 DataHub API keys

To make use of the Pathway content, you will require a DataHub account and a valid API key to access Polkadot via DataHub's infrastructure.
You will need to [sign up for a DataHub account](https://auth.figment.io/sign_up) and verify your email address.

To use your API key you must create a new file named `.env.local` in the project root directory `/learn-web3-dapp/`, copying the contents of the existing `.env.example` file. Your API key needs to be pasted into `.env.local` so that you can authenticate your connections with DataHub.

Your personal API key can be found on the [**DataHub Services Dashboard**](https://datahub.figment.io/). Click on the **Polkadot** icon in the list of available protocols and then copy your key as shown below:

![](../../../.gitbook/assets/pathways/polkadot/polkadot-setup.gif)

You can then paste your personal API key into `.env.local`, as the value for the environment variable `DATAHUB_POLKADOT_API_KEY`. This will authenticate you and enable you to make RPC requests to Polkadot via DataHub:

![API keys](https://user-images.githubusercontent.com/2707197/136940513-e1f95d43-f107-43ab-8cc4-bd0485510dbc.png)

---------------------------

{% hint style="info" %}
[**Join us on Discord**](https://figment.io/devchat), if you encounter any issues with the tutorial or have any questions!
{% endhint %}

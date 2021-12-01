# 🧩 DataHub API keys

To make use of the Pathway content, you will require a DataHub account and a valid API key to access Avalanche via DataHub's infrastructure.
You will need to [sign up for a DataHub account](https://auth.figment.io/sign_up) and verify your email address.

To use your API key you must create a new file named `.env.local` in the project root directory `/learn-web3-dapp/`, copying the contents of the existing `.env.example` file. Your API key needs to be pasted into `.env.local` so that you can authenticate your connections with DataHub.

Your personal API key can be found on the [**DataHub Services Dashboard**](https://datahub.figment.io/). Click on the **Avalanche** icon in the list of available protocols and then copy your key as shown below:

![](../../../.gitbook/assets/pathways/avalanche/avalanche-setup.gif)

You can then paste your personal API key into `.env.local`, as the value for the environment variable `DATAHUB_AVALANCHE_API_KEY`. This will authenticate you and enable you to make JSON-RPC requests to Avalanche via DataHub:

![API keys](https://user-images.githubusercontent.com/2707197/136942268-96486e9c-ce22-480f-a094-0b043e29d927.png)

---------------------------

{% hint style="info" %}
[**Join us on Discord**](https://figment.io/devchat), if you encounter any issues with the tutorial or have any questions!
{% endhint %}

The following software is required to set up and complete the **The Graph** Pathway

- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed
- [Node.js](https://nodejs.org/) v14.17.0 or higher installed
- [yarn](https://yarnpkg.com/getting-started/install) installed
- [docker](https://www.docker.com) installed

Start by cloning the repository and installing the dependencies with yarn :

```text
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

---

# Get an alchemy API-KEY

We could have used [public Ethereum RPC's endpoints](https://docs.linkpool.io/docs/public_rpc), but we have chosen to rely on [alchemy](https://www.alchemy.com/). Alchemy provides one of the leading blockchain development platform, and thus it's provide endpoints to Ethereum mainnet. We need one as later our local graph node will be listen on event on the Ethereum mainnet.

To get your alchemy API key you:

- Go to [alchemy](https://www.alchemy.com/)
- Create a Free account
- Create a simple dapps.
- Copy and paste your API key in safe place.

![](../assets/graph-alchemy-setup.gif)

---

# Start the server

With the API key in place, start the Next.js development server with:

```bash
yarn dev
```

Once the development server loads and compiles the application, open your default browser and navigate to `http://localhost:3000`:

![](../../../.gitbook/assets/pathway-home.gif)

{% hint style="warning" %}
Did you know you can change the port? By default Next.js uses port 3000, but if you have another service already running on that port, use the `--port` flag, like `yarn dev --port 1122`.
{% endhint %}

---

# Next

You can now move ahead to the next step by clicking on the "Next" button below on the right.

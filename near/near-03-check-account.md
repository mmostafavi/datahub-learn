Unlike many other Web 3 protocols, NEAR uses a human readable account ID instead of a public key hash. You can link as many `keypairs` to a NEAR account as you want. Here, we're going to Learn how to check the availability of a NEAR account name. As you might expect, **figment.testnet** is already taken.

---

# Challenge

{% hint style="tip" %}
In `pages/api/near/check-account.ts`, implement the default function. You must replace any instances of `undefined` with working code to accomplish this.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
  try {
    const { freeAccountId, NETWORK } = req.body
    const config = configFromNetwork(NETWORK);
    const near = await connect(config);
    // Query the account info of freeAccountId
    const accountInfo = undefined
    try {
        undefined;
        return res.status(200).json(false)
    } catch (error) {
        return res.status(200).json(true)
    }
  }
```

**Need some help?** Check out these links

- [The `Account` class](https://near.github.io/near-api-js/classes/account.account-1.html)
- [An explanation of `NEAR Accounts`](https://docs.near.org/docs/concepts/account)
- [RPC `view_account`](https://docs.near.org/docs/develop/front-end/rpc#view-account)

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

---

# Solution

```typescript
// solution
  try {
    const { freeAccountId, NETWORK } = req.body
    const config = configFromNetwork(NETWORK);
    const near = await connect(config);
    const accountInfo = await near.account(freeAccountId);
    try {
        await accountInfo.state();
        return res.status(200).json(false)
    } catch (error) {
        return res.status(200).json(true)
    }
  }
```

**What happened in the code above?**

- First, we create an `Account` object from the `freeAccountId` being passed in the request body.
- Next, we query the state of this object with the `state` method:
  - If it returns `true`, the account exists and we will return a `false` value to the client-side - indicating that the name is unavailable.
  - If `state` returns `false`, the account doesn't exist - so we pass a `true` value back to the client-side, indicating that the name is available. Phew! Little bit of programming logic there.

---

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route:

- Choose an account Id.
- Click on **Check it!**
  You should see:

![](../../../.gitbook/assets/pathways/near/near-account.gif)

---

# Conclusion

Great, now that you know how to test the availability of a chosen account name, it's time to create it. Let's do this in the next step!

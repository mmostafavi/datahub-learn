Unlike many other Web 3 protocols, NEAR uses a human readable account ID instead of a public key hash. You can link as many `keypairs` to a NEAR account as you want. Here, we're going to learn how to create an account on NEAR.

---

# Challenge

{% hint style="tip" %}
In `pages/api/near/create-account.ts`, implement the default function. You must replace any instances of `undefined` with working code to accomplish this.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
try {
    const { freeAccountId, publicKey, NETWORK } = req.body;
    const config = configFromNetwork(NETWORK);
    const near = await connect(config);
    undefined;
    return res.status(200).json(freeAccountId);
}
```

**Need some help?**

- [`createAccount` method](https://near.github.io/near-api-js/classes/near.near-1.html#createaccount)

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

---

# Solution

```typescript
// solution
try {
    const { freeAccountId, publicKey, NETWORK }  = req.body;
    const config = configFromNetwork(NETWORK);
    const near = await connect(config);
    await near.createAccount(freeAccountId, publicKey);
    return res.status(200).json(freeAccountId);
}
```

**What happened in the code above?**

- First, we need to [destructure](https://dmitripavlutin.com/javascript-object-destructuring/) the values from the request body so that we can use them in our code. We are also specifying a TypeScript type of `AccountReq` here.
- Then we use `configFromNetwork`, passing the `network` from the request body - now we can create a connection, `near`.
- Next, we call the `createAccount` method passing the `freeAccountId` and the `publicKey` from the request body.
- Finally, we can return the name of the account to the client-side as JSON.

---

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route.

Now click on **Create Account** and you should see:

![](../../../.gitbook/assets/pathways/near/near-account.gif)

---

# Conclusion

Every new account created on the testnet is given a free **airdrop** of 200 NEAR tokens. So cool!
Ready to move on? Let's check the account balance in the next step.

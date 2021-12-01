---
description: Learn how to setup your NEAR wallet and store your tokens
---

# 💼 NEAR Wallet

## Getting started with the NEAR wallet

[**The original post can be found here**](https://near.org/blog/getting-started-with-the-near-wallet/).

The [**NEAR Wallet**](https://wallet.near.org/) is a non-custodial, web-based wallet for the NEAR blockchain.

The wallet is now available to everyone! To get started, go to [**https://wallet.near.org**](https://wallet.near.org/) and click “Create Account”.

## Creating an Account

First, choose your account name. Each account created in the NEAR Wallet is appended by `.near`, which is part of the account name. For example, choosing `satoshi` will create the `satoshi.near` account name.

![Create Account](https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%286%29%20%281%29.png)

## Choosing an Access Method

Next, you will choose your account access method. The NEAR Wallet is entirely non-custodial, which means account access is your responsibility. We currently offer four methods of account access. After creating your account, you can choose to enable multiple methods.

![Protect Account](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-08-30-at-1.40.11-PM-1024x614.png)

### Ledger Hardware Wallet

If you have a Ledger Nano S or X, we **highly** recommend using it! This ensures your private keys never leave your Ledger, which provides the highest level of security when using the NEAR Wallet.

![Connect Ledger](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-08-13-at-3.52.10-PM-1024x595.png)

### Recovery \(Seed\) Phrase

Write down a twelve seed word recovery phrase. If you do not have a Ledger, this is the next best option.

**Important**: The seed phrase is only as secure as your storage of it! If you want to retain access to your account, you **MUST** write down your seed words in order, and store them securely. If you lose your seed words, you lose your account forever.

### ![Seed Phrase](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-08-13-at-3.52.27-PM-1024x629.png)

### Email/Phone

This is the least secure option. We only recommend this option for smaller amounts, as the security depends on your mobile carrier or email provider.

We will send you a one-time email or SMS message with a verification code and recovery link. We do not store this information, and thus you MUST keep this message safe in order to recover your account.

**Important**: We can only resend the recovery message, or update your chosen email or phone number, while you have access to your account. You must ensure you retain access to this message, otherwise, you will be unable to recover your account.

## Funding Your Account

Next, you need to fund your account with NEAR.

To do anything with the NEAR Wallet, you will need at least 2 NEAR. There are several ways to obtain NEAR, including from popular exchanges like Binance and Huobi.

To complete this step, transfer NEAR \(minimum 2 NEAR\) to the 64 character account ID \(your temporary ID\), then return to this page.

![Fund Account with NEAR](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-10-19-at-4.31.31-PM-1024x614.png)

After funding your account, you will see the “Claim Account” button.

Just click the button, and your account is ready to use! You will be redirected to your “Profile” page and can view your account details.

## Managing Your Account

On the “Profile” page, you will see a breakdown of your account balance, available recovery options \(not including your seed phrase from the first step\), and the option to add a Ledger hardware device or setup Two Factor Authentication.

![](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-10-19-at-4.33.22-PM-1024x599.png)

### Two Factor Authentication

If you do not have a Ledger, we highly recommend enabling Two Factor Authentication.

Two Factor Authentication in the NEAR Wallet deploys a multi-signature contract to your account, and requires all transactions to be confirmed by a second device.

We currently offer Email or SMS-based Two Factor Authentication.

**Note**: If you setup Ledger, you will not see the 2FA screen. We will eventually offer support for Ledger to be used as one of the authentication methods, but that is currently not possible in the Wallet.

#### ![Enable Two-Factor Authentication](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-08-13-at-3.56.14-PM-1024x618.png)

## Staking

One of the best ways to put your NEAR tokens to use is to stake them! Staking helps secure the NEAR network and will generate returns on your NEAR tokens. Based on the current total stake \(250 million NEAR staked\), you could earn returns of over 15% per year on your NEAR tokens. If the total stake increases, this rate will go down, and if it decreases, it will go up.

To stake your NEAR tokens, click on “Staking” in the navigation bar.

![Staking dashboard](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-10-23-at-7.58.27-AM-1024x638.png)

In the NEAR wallet, you can choose to stake your tokens with a validator. These validators run the infrastructure that powers the NEAR network and will do the work on your behalf to generate returns on your NEAR. For this work, most will charge a fee, which is only charged on your rewards \(not your total stake\).

Click “Select Validator”, then choose one from the list \(or search for one if you have one in mind\). A complete list with further details of each validator can be found on [https://near-staking.com](https://near-staking.com).

![Select a Validator](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-10-23-at-8.20.53-AM-1024x643.png)

Once you’ve identified a validator you want to stake with, click “Stake” next to their name, then enter the amount of NEAR to stake.

After staking, you will be returned to the dashboard. Here, you can view a summary of your staking positions, unclaimed rewards, and any tokens that are in the unstaking process or ready to withdraw. Scroll down to see a list of all validators you are actively staking with, and how much NEAR is staked with them.

### Unstaking

To unstake, select the validator from “Current Validators” you would like to unstake from. This will take you to the validator summary page. Click “Unstake” to initiate the unstaking process. Unstaking takes 36-48 hours \(3 full epochs\), and your tokens will still be unavailable during this time \(they will show as “Pending Release”\).

![Validator&apos;s Page](https://near.org/wp-content/uploads/2020/09/Screen-Shot-2020-10-23-at-8.22.32-AM-1024x642.png)

Once 3 epochs have completed, you will see them in “Ready to Withdraw” on the dashboard. Select the validator’s name again, and click the “Withdraw” button to withdraw your tokens from the validator’s account to your own account.

# Next Steps

Congratulations, you’ve setup a NEAR account, funded it, and successfully staked your NEAR! We will release more tutorials as we add additional functionality to the Wallet.


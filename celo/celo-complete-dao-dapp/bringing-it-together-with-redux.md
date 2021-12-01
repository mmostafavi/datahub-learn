---
title: Build a Decentralized Autonomous Organization (DAO) on Celo - Part 4
description: Learn how to create a functional DAO by examining the smart contract code in detail, then assembling a React Native app which interacts with the smart contract on Celo.
---

# Build a Decentralized Autonomous Organization (DAO) on Celo - Part 4

In this tutorial, we will complete our DAO and its dApp interface, writing the Redux code to connect our React Native app to the smart contract on Celo.
# Prerequisite

[Redux](https://redux.js.org/introduction/getting-started)

&nbsp;

In this section, we will look into how to connect our mobile app to the smart contract on the network. This section contains redux action creators for connecting to the contract, contributing to the DAO and so forth.

Here’s the action creator to contribute some certain amount of celo to the dapp:

```javascript
function contribute(amount) {
  return async (dispatch) => {
    dispatch(request());


    try {
      const goldToken = await kit.contracts.getGoldToken();
      const contractAddress = await (await contractInstance).options.address
      const requestedAmount = web3.utils.toWei(amount, 'ether');
      const txObject = goldToken.transfer(contractAddress, requestedAmount).txo;


      const requestId = 'contribution';
      const dappName = 'Charlo';
      const callback = Linking.makeUrl('ProfilePage');


      requestTxSig(
        kit,
        [
          {
            tx: txObject,
            from: kit.defaultAccount,
            to: goldToken.contract.options.address,
            feeCurrency: FeeCurrency.cUSD
          }
        ],
        { requestId, dappName, callback }
      );


      const response = await waitForSignedTxs(requestId);
      const rawTx = response.rawTxs[0];


      const tx = await kit.connection.sendSignedTransaction(rawTx);
      const receipt = await tx.waitReceipt();


      receipt.daoBalance = web3.utils.fromWei((await goldToken.balanceOf(contractAddress)).toString(), 'ether');
      receipt.userBalance = web3.utils.fromWei((await goldToken.balanceOf(kit.defaultAccount)).toString(), 'ether');


      dispatch(success(receipt));
      dispatch(alertActions.success("Contribution successful"));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed());
    }
  };


  function request() { return { type: profileConstants.CONTRIBUTION_REQUEST } };
  function success(res) { return { type: profileConstants.CONTRIBUTION_SUCCESS, res } };
  function failed() { return { type: profileConstants.CONTRIBUTION_FAILED } };
}
```

This is an async action that is made up of multiple sub actions, this is because we have to make an http request and wait for the response before completing. Async actions typically dispatch a request action before performing an async task (e.g. an http request), and then dispatch a success or error action based on the result of the async task.

For example the contribute() action creator performs the following steps:
Dispatches a CONTRIBUTION_REQUEST action with dispatch(request());
Wire up the necessary calls to setup the transaction from line 6 - 29
Sends the transaction on line 31 with an async call to sendSignedTransaction(rawTx)
Dispatches a CONTRIBUTION_SUCCESS action with dispatch(success(receipt)); if contribution was successful, or dispatches a CONTRIBUTION_FAILED action with dispatch(failed()); if login failed.

To keep the code tidy, sub action creators are nested within each async action creator function. For example the contribue() function contains 3 nested action creator functions for request(), success() and failure() that return the actions CONTRIBUTION_REQUEST, CONTRIBUTION_SUCCESS and CONTRIBUTION_FAILURE respectively. Putting the sub action creators into nested functions also allows us to give them standard names like request, success and failure without clashing with other function names because they only exist within the scope of the parent function.

All other action creators follow the same process as the one explained above.
Connection Action Creators with The UI:



Profile Screen

```javascript
import * as React from 'react';
...
import { profileActions } from '../store/actions';
...

export const ProfilePage = ({ navigation }) => {
...
  const dispatch = useDispatch();
  const profile = useSelector(state => state.profile);
...

  const contribute = async () => {
    const contributed = amountInput.value;
    if (contributed === '') {
      return;
    }

    await dispatch(profileActions.contribute(contributed));
...

  };

...
  return (
    ...
          
          <Button
            size='small'
            onPress={contribute}
            accessoryLeft={profile.loading ? loadingIndicator : ''}
            >
            CONTRIBUTE
          </Button>
...

  );
}
```

Conclusion
In this tutorial, you learned how to build a complete Charity DAO. You wrote the complete smart contract code using the Solidity programming language. Built the App using React Native and then used Redux to tie the actions together!
The Smart Contract code is deployed on the Alfajores testnet. The contract allows the members to contribute to the DAO. 

The [first part](./introduction) of the tutorial serves as the foundation which explains what the rest of the project is about. It opened by explaining what is a Decentralized Autonomous Organization and the objective for building one. Then, it we went ahead to explain the required tools to build the DAO. 

In [part 2](./part-2) we wrote the Smart Contract code using Solidity and Truffle. Then we explained each of the functions.

In [part 3](./part-3) we built a React Native app to communicate with the Solidity smart contract on Celo.

[Part 4](./part-4) is the concluding section where we tied the React Native App and the Smart Contract using Redux.

The App is available in an Open Source code repository on [Github](https://github.com/PhoenixTechAfrica/Tutorials/tree/dev/projects/charlo). You are encourged to fork it and use it as a source to compare with your own code. We will also be glad if you can leave a star on it! ⭐️

We hope this was not the longest tutorial for you to follow. Lol. Congratulations if you have made it this far! There’s no limits to what you can build given the knowledge that you have thus acquired reading and following this tutorial! Go forth and build great things! 

About the Authors
This tutorial was created by [Segun Ogundipe](https://community.figment.io/u/segun-ogundipe). Connect with Segun on [LinkedIn](https://www.linkedin.com/in/segun-ogundipe/) and [Emmanuel Oaikhenan](https://community.figment.io/u/odia.emma/).
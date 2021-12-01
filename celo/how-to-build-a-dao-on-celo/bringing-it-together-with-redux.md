# Bringing it together with Redux

In this tutorial, we will complete our DAO and its dApp interface, writing the Redux code to connect our React Native app to the smart contract on Celo. We will look into how to connect the mobile app to the smart contract on the Celo network. This section contains Redux action creators for connecting to the contract, contributing to the DAO and so forth.  
  
If you are unfamiliar with Redux, please read their [Getting Started](https://redux.js.org/introduction/getting-started) guide.

Here’s the action creator to contribute some certain amount of Celo to the dApp:

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

This is an asynchronous action that is made up of multiple sub actions. This is because we have to make an http request and wait for the response before completing. Async actions typically dispatch a request action before performing an async task \(e.g. an http request\), and then dispatch a success or error action based on the result of the async task.

For example the `contribute()` action creator performs the following steps:

* Dispatches a `CONTRIBUTION_REQUEST` action with `dispatch(request());` 
* Wire up the necessary calls to setup the transaction 
* Sends the transaction with an async call to `sendSignedTransaction(rawTx)` 
* Dispatches a `CONTRIBUTION_SUCCESS` action with `dispatch(success(receipt));` if contribution was successful, or dispatches a `CONTRIBUTION_FAILED` action with `dispatch(failed());` if login failed.

To keep the code tidy, sub action creators are nested within each async action creator function.   
For example the `contribute()` function contains 3 nested action creator functions for `request()`, `success()` and `failure()` that return the actions `CONTRIBUTION_REQUEST`, `CONTRIBUTION_SUCCESS` and `CONTRIBUTION_FAILURE` respectively.   
Putting the sub action creators into nested functions also allows us to give them standard names like request, success and failure without clashing with other function names because they only exist within the scope of the parent function.

All other action creators for this project follow the same process as the one explained above. 

## Connecting Action Creators with The UI

{% hint style="info" %}
Three dots `...` on a line by themselves indicate the presence of other code which we have trimmed for display purposes.
{% endhint %}

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

# Conclusion

In this tutorial, you learned how to build a complete Charity DAO. We covered the complete smart contract code using the Solidity programming language. We scaffolded the App using React Native and then used Redux to tie the actions together!   
  
The Smart Contract code is deployed on the Alfajores testnet. The contract allows the members to contribute to the DAO.

The App is available in an Open Source code repository on [Github](https://github.com/PhoenixTechAfrica/Tutorials/tree/dev/projects/charlo). You are encouraged to fork it and use it as a starting point for your own DAO.

Congratulations for completing these tutorials. There are no limits to what you can build using the tools and techniques we have discussed.

# About the authors 

This tutorial was created by [Segun Ogundipe](https://community.figment.io/u/segun-ogundipe) and [Emmanuel Oaikhenan](https://community.figment.io/u/odia.emma/).


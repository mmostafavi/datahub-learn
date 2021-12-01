# Building the React Native App

Note: If you have never built an app using React Native to connect to the Celo wallet before, you can follow the guide [here](https://learn.figment.io/tutorials/how-to-successfully-connect-to-a-celo-wallet-with-a-react-native-dapp#project-setup).

In this tutorial, we will continue building a functional DAO by making a React Native app to communicate with the Solidity smart contract on Celo. We will make use of the [UI Kitten](https://akveo.github.io/react-native-ui-kitten/) library to style the dApp.  
  
This is an outline of the Pages and Components that make up the dApp. The next section on Redux will be a deep dive into how to connect the dApp to the smart contract.

First, we will outline the UI and then write out the code for building each of the views:

* Welcome Screen \(Connect Wallet\)
* Side Menu Navigation
* All Proposals Screen 
* Votable Proposals \[List\]
* Create Proposal \[Modal, Form\]
* View Proposal \[Modal\]
* Upvote \[Button\]
* Downvote \[Button\]
* Profile Screen
* Contribute to the DAO \[Modal, Form\]
* Proposals voted on \[List\]
* View Proposal \[Modal\]
* Payable Proposals \[List\]
* View Proposal \[Modal\]
* Make Payment \[Button\] 
* Paid Proposals \[List\]
* View Proposal \[Modal\]
* About

At this stage of making the dApp, we will outline how the actions of the smart contract are carried out with a connection to the dApp. First, let’s put the code in place for all the screens. The screens at this point do not yet interact with the smart contract.

## Initialize the dApp

This section assumes that you already know how to initialize and start a React Native App. This will serve as a high level overview on how to start your React Native app.

### Setting up UI Kitten

UI Kitten is a customizable React Native UI Library based on Eva Design System specifications, it is a framework of UI components that can easily be added to a React Native app.   
Initializing the app with the `expo init` command and the blank managed workflow as the selected template sets up a basic @ui-kitten/components configuration for us. If this does not work for you, install UI Kitten using the following command instead:

```text
yarn add @ui-kitten/components @eva-design/eva react-native-svg
```

Once the installation process is complete, you can use any of the components available in the UI Kitten library. First, we have to configure the UI Kitten library at the root of the application, before its components can be called anywhere in the dApp.

Open the `App.js` file and wrap the project in UI Kitten’s `ApplicationProvider` module and also add the `IconRegistry` module so we can use @ui-kitten/eva-icons Icons.

{% code title="App.js" %}
```javascript
import './global';
import * as eva from '@eva-design/eva';
import { ApplicationProvider, IconRegistry } from '@ui-kitten/components';
import { EvaIconsPack } from '@ui-kitten/eva-icons';
import { StatusBar } from 'expo-status-bar';
import React from 'react';
import { Navigation } from './navigation'

export default function App() {
  const isLoadingComplete = useCachedResources();
  const colorScheme = useColorScheme();

  if (!isLoadingComplete) {
    return null;
  } else {
    return (      
        <ApplicationProvider
          {...eva}
          theme={{...eva.dark, ...theme}}
          customMapping={mapping}>
          <IconRegistry icons={EvaIconsPack} />
          <Navigation coloScheme={colorScheme} />
          <StatusBar />
        </ApplicationProvider>
    );
  }
}
```
{% endcode %}

We are making use of the eva dark theme and we have configured `IconRegistry` to use the `EvaIconsPack`. Now that we have UI Kitten set up, we can style our dApp as we build. Now let’s go ahead and build out the screens!

## Screens

### Welcome Screen

The Welcome screen is the first thing users will see when loading the dApp. The UI is only one button which the user must click to connect to the DAO. Since our app is making use of the Celo protocol, we will have to connect to the Alfajores/Valora wallet.

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2829%29.png)

Below is the code needed to create the **Welcome** screen:

```javascript
 export const WelcomePage = ({navigation}) => {

 return (
   <Layout style={styles.container}>
     <Button
       raised='true'
       accessoryLeft={profile.loading ? loadingIndicator : ""}
     >Connect To Wallet</Button>
     <Text style={styles.text}>
       Welcome to the Charity DAO example
     </Text>
   </Layout>
 );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  text: {
    paddingTop: 10,
  },
  indicator: {
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

{% hint style="warning" %}
Notice anything missing? Don't worry, we will add the functionality to connect this screen to the Wallet and carry out user actions in the Redux tutorial.
{% endhint %}

### Side Menu Navigation

This is based on React Navigation’s Drawer Navigation. To use the React Drawer navigation, you have to install and set up React Navigation following the instructions in the [docs](https://reactnavigation.org/docs/getting-started). The instructions for designing a Drawer based navigation can be found [here](https://reactnavigation.org/docs/drawer-based-navigation).

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2828%29.png)

### All Proposals Screen

The All Proposals screen contains all the DAO proposals that have been created. The UI is a button at the top which will open a modal window containing a form to be filled out by the user creating the proposal.

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2830%29.png)

This is the code to create the All Proposals screen:

```javascript
export const ProposalsPage = ({ navigation }) => {
  const theme = useTheme();
  const chaFooter = (props, info) => {
    return(
      <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'space-evenly', padding: 8}}>
        <Text style={{color: theme['color-info-default']}}>For: {info.for}</Text>
        <Text style={{color: theme['color-danger-default']}}>Against: {info.against}</Text>
      </View>
    );
  };
  const cardItem = (info) => {
    return(
      <Card
        style={{borderColor: theme['color-primary-default'], marginVertical: 4}}
        footer={props => chaFooter(props, {for: 10, against: 3})}
        >

        <Text category='s2' numberOfLines={4} ellipsizeMode='tail'>{info.item.desc}</Text>
      </Card>
    );
  };

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <Layout style={{ flex: 1, padding: 16 }}>
        <Card
          disabled='true'
          style={{borderColor: theme['color-primary-default'], margin: 8, padding: 8}}>
          <Button size='medium'>Create Proposal</Button>
        </Card>
        <List
          contentContainerStyle={{paddingHorizontal: 8, paddingVertical: 4}}
          renderItem={cardItem}/>
      </Layout>
    </SafeAreaView>
  );
};
```

### Create Proposal Modal

The Create Proposal modal component is imported into the Proposals page and is displayed when the Create proposal Button is clicked.

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2826%29.png)

Below is the code snippet to create the modal:

```javascript
export const CreateProposalModal = ({setVisible, visible,}) => {
  const theme = useTheme();
  const cardHeader = (props) => {
    return(
      <View {...props} style={{...props.style, flex: 1, flexDirection: 'row', justifyContent: 'space-between'}}>
        <Text style={{margin: 16}} category='h6'>Create Proposal</Text>
        <Button
          size='large'
          onPress={() => setVisible(false)}
          appearance='ghost'
          accessoryLeft={closeIcon}/>     
      </View>
    );
  };
  const cardFooter = (props) => {
    return(
      <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'flex-end', padding: 8}}>
        <Button
          style={{marginHorizontal: 2}}
          size='small'
          status='basic'
          onPress={() => setVisible(false)}>
          CANCEL
        </Button>
        <Button
          style={{marginHorizontal: 2}}
          size='small'
          >
          CREATE
        </Button>
      </View>
    );
  };

  return(
    <Modal
      visible={visible}
      backdropStyle={{backgroundColor: theme['color-primary-transparent-300']}}
      onBackdropPress={() => setVisible(false)}
      >
      <Card
        disabled='true'
        style={{flex: 1, borderColor: theme['color-primary-default'], margin: 2}}
        header={cardHeader}
        footer={cardFooter}>
        <Layout style={{flex: 1, padding: 8, width: 250}}>
          <Input
            size='medium'
            style={{marginVertical: 8}}
            status='primary'
            keyboardType='numeric'
            placeholder='Enter amount'
            />
          <Input
            size='medium'
            style={{marginVertical: 8}}
            status='primary'
            placeholder='Enter Charity address'
            />
          <Input
            multiline={true}
            textStyle={{minHeight: 64}}
            style={{marginVertical: 8}}
            status='primary'
            placeholder='Enter the description'
            />
        </Layout>    
      </Card>
  </Modal>
  );
};

const closeIcon = (props) => {
  return(
    <Icon {...props} name='close-circle-outline'/>
  );
};
```

Before we move on to the Create Proposal screen, we can add data input and validation logic for the input fields on the form.

Add the following code snippet after `useTheme()`;

```javascript
const descriptionInput = useInputState('Description');
const charityAddressInput = useInputState('Address');
const amountInput = useInputState('Amount');
const addItem = () => {
  if (amountInput.value == ''
      || descriptionInput.value.length < 20
      || charityAddressInput.value == '') {
      return;
    }
    let newData = [...data];
    newData.push({
      amount: amountInput.value,
      desc: descriptionInput.value,
      chaAdd: charityAddressInput.value
    });
    amountInput.setValue('');
    descriptionInput.setValue('');
    charityAddressInput.setValue('');
    setData(newData);
    setVisible(false);
  };
```

On the Create Button, we can call the `addItem` function in the Button's `onPress` property to add an item.

```javascript
<Button onPress={addItem}>CREATE</Button>
```

On the input fields, pass the relevant variable using [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) to set the Input items:

```javascript
<Input
     ...            
     placeholder='Enter amount'
     {...amountInput}/>
<
     ...
     placeholder='Enter Charity address'
     {...charityAddressInput}/>
 <Input
      ...
      placeholder='Enter the description'
      {...descriptionInput}/>
```

The `useInputState()` method which the input fields were assigned is used for Data Validation. Add the code below after the close of the return block, before the `closeIcon()` function:

```javascript
const useInputState = (name, initialValue = '') => {
  const [value, setValue] = React.useState(initialValue);

  let caption;
  if (value === '') {
    caption = `${name} cannot be empty`;
  }

  if (name === 'Description' && value.length < 20) {
    caption = `${name} must be more than 20 characters`
  }

  return {value, onChangeText: setValue, caption, setValue};
};
```

After creating the Modal, we can then import it into the Create Proposal screen and establish the state of the modal, by adding states to toggle between `“visible”` and `“hidden”`. Here is how:

First import the `CreateProposal` module into the Proposal page:

```javascript
import { CreateProposalModal, ViewProposalModal } from '../components';
```

Using React's [useState hook](https://reactjs.org/docs/hooks-state.html), the state of the Create modal can be set to visible when it is clicked.

```javascript
const [createVisible, setCreateVisible] = React.useState(false);
```

In the return statement inside of `Layout` add the CreateProposalModal Component and set the state of the component.

```javascript
<CreateProposalModal
  setVisible={setCreateVisible}
  visible={createVisible}
/>
```

### Profile Screen

The Profile screen contains an input field through which users can contribute some Celo tokens \(at least 5 Celo\) to become a Contributor. Below are the static elements of the Profile screen. We will be adding the user actions in the Redux tutorial ahead.

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2825%29.png)

The Profile screen is completely styled and makes use of cards for the different sections. Examine the code below:

```javascript
import * as React from 'react';
import { View, SafeAreaView } from 'react-native';
import { Button, Card, Input, Layout, List, useTheme, Text, Spinner, ViewPager, Icon } from '@ui-kitten/components';

export const ProfilePage = ({ navigation }) => {  
  const theme = useTheme();
  const cardItem = (info) => {
    return(
      <Card
        style={{borderColor: theme['color-primary-default'], marginVertical: 4}}
        footer={props => cardFooter(props, {for: info.item[3], against: info.item[4]})}
        onPress={() => getProposal(info.item[0])}>
        <Text category='s2' numberOfLines={4} ellipsizeMode='tail'>{info.item[5]}</Text>
      </Card>
    );
  };

  const cardFooter = (props, info) => {
    return(
      <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'space-evenly', padding: 8}}>
        <Text style={{color: theme['color-info-default']}}>For: {info.for}</Text>
        <Text style={{color: theme['color-danger-default']}}>Against: {info.against}</Text>
      </View>
    );
  };

  const page = () => {
    return(
      <>
        <Layout style={{flexDirection: 'row', justifyContent: 'space-around'}}>
          <Button
            appearance='ghost'
            status='info'
            accessoryLeft={leftIcon}
            onPress={handleLeftClick}
            />
          <Text style={{alignSelf: 'center', marginVertical: 16}}>{selectedIndex == 0 ? 'In Voting' : selectedIndex == 1 ? 'Payable Proposals' : 'Paid Proposals'}</Text>
          <Button
            appearance='ghost'
            status='info'
            accessoryRight={rightIcon}
            onPress={handleRightClick}
            />
        </Layout>
        {
          (profile.loading || !profile.isStakeholder) ?
          <Text style={{backgroundColor: theme['color-basic-800']}}>{selectedIndex == 0 ? 'The list of "in voting" proposal is empty' : selectedIndex == 1 ? 'The list of "payable" proposal is empty' : 'The list of "paid" proposal is empty'}</Text> :
          <List
          style={{backgroundColor: theme['color-basic-800']}}
          contentContainerStyle={{paddingHorizontal: 8, paddingVertical: 4}}
          data={selectedIndex == 0 ? inVotingVotes : selectedIndex == 1 ? payableVotes : paidVotes}
          renderItem={cardItem}/>
        }
      </>
    );
  };

  const rightIcon = (props) => {
    if (selectedIndex != 2) {
      return(
        <Icon
          {...props}
          name='arrowhead-right-outline'/>
      );
    }

    return(null);
  };

  const leftIcon = (props) => {
    if (selectedIndex != 0) {
      return(
        <Icon
          {...props}
          name='arrowhead-left-outline'/>
      );
    }

    return(null);
  };

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <Layout style={{ flex: 1, padding: 16 }}>
        <Card style={{flexDirection: 'column', justifyContent: 'space-between', borderColor: theme['color-basic-800']}}>
          <Input
            style={{marginVertical: 8}}
            size='medium'
            status='primary'
            keyboardType='numeric'
            placeholder='Enter amount to contribute'
            {...amountInput}/>

          <Button
            size='small'
            onPress={contribute}
            accessoryLeft={profile.loading ? loadingIndicator : ''}
            >
            CONTRIBUTE
          </Button>
        </Card>

        <ViewPager style={{height: 480}}
          selectedIndex={selectedIndex}
          shouldLoadComponent={shouldLoadComponent}
          onSelect={setSelectedIndex}>

            <Layout level='2'>
              {page()}
            </Layout>

            <Layout level='2'>
              {page()}
            </Layout>

            <Layout level='2'>
              {page()}
            </Layout>
          </ViewPager>

      </Layout>
    </SafeAreaView>
  );
};
```

### View Proposal Modal

The View Proposal modal component is imported into the Profile page and is displayed when a Proposal Card is clicked.

![](../https://github.com/figment-networks/learn-tutorials/raw/master/assets/image%20%2831%29.png)

Below is the code snippet to create the modal:

```javascript
import * as React from 'react';
import { View } from 'react-native';
import { Button, Card, Icon, Input, Layout, Modal, Text, useTheme, Spinner } from '@ui-kitten/components';

export const ViewProposalModal = ({setVisible, visible }) => {
  const theme = useTheme();
  const cardHeader = (props) => {
    return(
      <View {...props} style={{...props.style, flex: 1, flexDirection: 'row', justifyContent: 'space-between'}}>
        <Text style={{margin: 16}} category='h6'>Proposal</Text>
        <Button
          size='large'
          onPress={() => setVisible(false)}
          appearance='ghost'
          accessoryLeft={closeIcon}/>
      </View>
    );
  };

  const cardFooter = (props) => {
          return(
            <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'space-around', padding: 8}}>
              <Button
                style={{marginHorizontal: 2}}
                size='small'
                status='success'
                >
                UPVOTE
              </Button>
              <Button
                style={{marginHorizontal: 2}}
                size='small'
                status='warning'>
                DOWNVOTE
              </Button>
            </View>
          );
  };

  return(
    <Modal
      visible={visible}
      backdropStyle={{backgroundColor: theme['color-primary-transparent-300']}}
      onBackdropPress={() => setVisible(false)}>
      <Card
        disabled='true'
        style={{flex: 1, borderColor: theme['color-primary-default'], margin: 2}}
        header={cardHeader}
        footer={cardFooter}> 
            <>
              <Layout style={{flex: 1, width: 250}}>
                <Text style={{marginVertical: 8}}>Description: </Text>
                <Text style={{marginVertical: 8}} category='p2'>Requested:  Celo</Text>
                <Text style={{marginVertical: 8}} category='p2'>Proposer: </Text>
                <Text style={{marginVertical: 8}} category='p2'>Charity: </Text>
              </Layout>
              <Layout style={{flex: 1, flexDirection: 'row', justifyContent: 'space-around'}}>
                <Text style={{color: theme['color-success-default']}}>For: </Text>
                <Text style={{color: theme['color-danger-default']}}>Against: </Text>
              </Layout>
            </>
      </Card>
    </Modal>
  );
};

const closeIcon = (props) => {
  return(
    <Icon {...props} name='close-circle-outline'/>
  );
};
```

# Next Steps

All of our app pages are now complete!   
  
To be able to interact with the smart contract on Celo, we will use Redux actions and reducers to connect with the functions in the smart contract. The next tutorial covers Redux and how to make these connections between actions performed by users and the smart contract functionality.


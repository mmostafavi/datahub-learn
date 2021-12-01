---
title: Build a Decentralized Autonomous Organization (DAO) on Celo Part 2
description: Learn how to create a functional DAO by examining the smart contract code in detail, then assembling a React Native app which interacts with the smart contract on Celo.
---
# Build a Decentralized Autonomous Organization (DAO) on Celo - Part 2

In this tutorial, we are going to build a functional DAO by writing the smart contract code, then building a React Native app to interact with the smart contract on Celo.


&nbsp;

# Coding the DAO Smart Contract
The smart contract is written using the Solidity programming language. Solidity is used on Ethereum as well as other Ethereum Virtual Machine (EVM) compatible blockchains like Celo.

# Initial Setup 

The first task is to initialize a basic Solidity project using truffle. Running the command `truffle init` in our preferred location - most commonly an empty directory somewhere under the users home directory - will install the default files for the project. After running this command, truffle will set up a basic solidity project with the necessary files.

# Code

Here is an overview of the full code for the DAO contract, we’ll go over the different functions below to look at what they do and why they are needed.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract CharloDAO is ReentrancyGuard, AccessControl {
    bytes32 public constant CONTRIBUTOR_ROLE = keccak256("CONTRIBUTOR");
    bytes32 public constant STAKEHOLDER_ROLE = keccak256("STAKEHOLDER");
    uint32 constant minimumVotingPeriod = 1 weeks;
    uint256 numOfProposals;

    struct CharityProposal {
        uint256 id;
        uint256 amount;
        uint256 livePeriod;
        uint256 votesFor;
        uint256 votesAgainst;
        string description;
        bool votingPassed;
        bool paid;
        address payable charityAddress;
        address proposer;
        address paidBy;
    }

    mapping(uint256 => CharityProposal) private charityProposals;
    mapping(address => uint256[]) private stakeholderVotes;
    mapping(address => uint256) private contributors;
    mapping(address => uint256) private stakeholders;

    event ContributionReceived(address indexed fromAddress, uint256 amount);
    event NewCharityProposal(address indexed proposer, uint256 amount);
    event PaymentTransfered(
        address indexed stakeholder,
        address indexed charityAddress,
        uint256 amount
    );

    modifier onlyStakeholder(string memory message) {
        require(hasRole(STAKEHOLDER_ROLE, msg.sender), message);
        _;
    }

    modifier onlyContributor(string memory message) {
        require(hasRole(CONTRIBUTOR_ROLE, msg.sender), message);
        _;
    }

    function createProposal(
        string calldata description,
        address charityAddress,
        uint256 amount
    )
        external
        onlyStakeholder("Only stakeholders are allowed to create proposals")
    {
        uint256 proposalId = numOfProposals++;
        CharityProposal storage proposal = charityProposals[proposalId];
        proposal.id = proposalId;
        proposal.proposer = payable(msg.sender);
        proposal.description = description;
        proposal.charityAddress = payable(charityAddress);
        proposal.amount = amount;
        proposal.livePeriod = block.timestamp + minimumVotingPeriod;

        emit NewCharityProposal(msg.sender, amount);
    }

    function vote(uint256 proposalId, bool supportProposal)
        external
        onlyStakeholder("Only stakeholders are allowed to vote")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];

        votable(charityProposal);

        if (supportProposal) charityProposal.votesFor++;
        else charityProposal.votesAgainst++;

        stakeholderVotes[msg.sender].push(charityProposal.id);
    }

    function votable(CharityProposal storage charityProposal) private {
        if (
            charityProposal.votingPassed ||
            charityProposal.livePeriod <= block.timestamp
        ) {
            charityProposal.votingPassed = true;
            revert("Voting period has passed on this proposal");
        }

        uint256[] memory tempVotes = stakeholderVotes[msg.sender];
        for (uint256 votes = 0; votes < tempVotes.length; votes++) {
            if (charityProposal.id == tempVotes[votes])
                revert("This stakeholder already voted on this proposal");
        }
    }

    function payCharity(uint256 proposalId)
        external
        onlyStakeholder("Only stakeholders are allowed to make payments")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];

        if (charityProposal.paid)
            revert("Payment has been made to this charity");

        if (charityProposal.votesFor <= charityProposal.votesAgainst)
            revert(
                "The proposal does not have the required amount of votes to pass"
            );

        charityProposal.paid = true;
        charityProposal.paidBy = msg.sender;

        emit PaymentTransfered(
            msg.sender,
            charityProposal.charityAddress,
            charityProposal.amount
        );

        return charityProposal.charityAddress.transfer(charityProposal.amount);
    }

    receive() external payable {
        emit ContributionReceived(msg.sender, msg.value);
    }

    function makeStakeholder(uint256 amount) external {
        address account = msg.sender;
        uint256 amountContributed = amount;
        if (!hasRole(STAKEHOLDER_ROLE, account)) {
            uint256 totalContributed =
                contributors[account] + amountContributed;
            if (totalContributed >= 5 ether) {
                stakeholders[account] = totalContributed;
                contributors[account] += amountContributed;
                _setupRole(STAKEHOLDER_ROLE, account);
                _setupRole(CONTRIBUTOR_ROLE, account);
            } else {
                contributors[account] += amountContributed;
                _setupRole(CONTRIBUTOR_ROLE, account);
            }
        } else {
            contributors[account] += amountContributed;
            stakeholders[account] += amountContributed;
        }
    }

    function getProposals()
        public
        view
        returns (CharityProposal[] memory props)
    {
        props = new CharityProposal[](numOfProposals);

        for (uint256 index = 0; index < numOfProposals; index++) {
            props[index] = charityProposals[index];
        }
    }

    function getProposal(uint256 proposalId)
        public
        view
        returns (CharityProposal memory)
    {
        return charityProposals[proposalId];
    }

    function getStakeholderVotes()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256[] memory)
    {
        return stakeholderVotes[msg.sender];
    }

    function getStakeholderBalance()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256)
    {
        return stakeholders[msg.sender];
    }

    function isStakeholder() public view returns (bool) {
        return stakeholders[msg.sender] > 0;
    }

    function getContributorBalance()
        public
        view
        onlyContributor("User is not a contributor")
        returns (uint256)
    {
        return contributors[msg.sender];
    }

    function isContributor() public view returns (bool) {
        return contributors[msg.sender] > 0;
    }
}
```
&nbsp;

Let’s go ahead and dissect the different functions that make up the smart contract.

&nbsp;

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
```
The opening line of a Solidity file should contain the [SPDX licence](https://spdx.org/licenses/) identifier of the relevant open source license - commonly this will be MIT or The Unlicense.
The next line will specify the Solidity version needed to compile the contract, using the `pragma` [compiler directive](https://docs.soliditylang.org/en/v0.5.8/layout-of-source-files.html#version-pragma). Be aware of the [semantic versioning](https://semver.org/).

&nbsp;
```
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
```
We’ll be relying on [OpenZepellin's](https://openzeppelin.com/) ReentrancyGuard and AccessControl here so we should go ahead and import those.

&nbsp;
```
bytes32 public constant CONTRIBUTOR_ROLE = keccak256("CONTRIBUTOR");
bytes32 public constant STAKEHOLDER_ROLE = keccak256("STAKEHOLDER");
```
Users of the DAO will be of two types - Contributors and Stakeholders. We need to declare two constants here, which are the [keccak256 hash](https://docs.soliditylang.org/en/v0.8.6/units-and-global-variables.html#mathematical-and-cryptographic-functions) of the words themselves. These constants will be used later to register and differentiate users.

&nbsp;
```
uint32 constant minimumVotingPeriod = 1 weeks;
```
The `minimumVotingPeriod` variable holds the number of days a proposal can be voted on in UNIX time. The [time unit](https://docs.soliditylang.org/en/v0.8.6/units-and-global-variables.html#time-units) weeks is a suffix provided by Solidity, 1 weeks here translates to the total seconds of [UNIX time](https://en.wikipedia.org/wiki/Unix_time) a week from now.
The value of UNIX time can be displayed with the command date +%s.%N in Linux. macOS users need to run brew install coreutils to enable microseconds display and use gdate +%s.%N instead.

&nbsp;
```
uint256 numOfProposals;
```
`numOfProposals` is incremented every time a new charity proposal is added. This lets us iterate through the charity proposals in the mapping data type, as Solidity doesn’t provide a way to step through mappings.

&nbsp;
```
    struct CharityProposal {
        uint256 id;
        uint256 amount;
        uint256 livePeriod;
        uint256 votesFor;
        uint256 votesAgainst;
        string description;
        bool votingPassed;
        bool paid;
        address payable charityAddress;
        address proposer;
        address paidBy;
    }
```
The `CharityProposal` struct definition holds the necessary data that makes up each proposal object.

&nbsp;
```
    mapping(uint256 => CharityProposal) private charityProposals;
    mapping(address => uint256[]) private stakeholderVotes;
    mapping(address => uint256) private contributors;
    mapping(address => uint256) private stakeholders;
```

&nbsp;

There are four mappings used:
- `charityProposals` maps a uint256 value and a CharityProposal as key and value respectively. This holds the list of Proposals in the DAO. It uses the id of the Proposal as key and the Proposal itself as the value.
- `stakeholderVotes` maps the `address` of a Stakeholder to a list of the Proposals that address has voted on.
- `contributors` maps the Contributor addresses and the amounts they have sent into the DAO treasury.
- `stakeholders` maps the addresses and balances of Stakeholders.

```
    event ContributionReceived(address indexed fromAddress, uint256 amount);
    event NewCharityProposal(address indexed proposer, uint256 amount);
    event PaymentTransfered(
        address indexed stakeholder,
        address indexed charityAddress,
        uint256 amount
    );
```

These events are emitted for every new proposal, new contribution and new payment transfer. This is for logging purposes and to make filtering these events simpler. The indexed attribute causes the respective arguments to be treated as log topics instead of data.

```
    modifier onlyStakeholder(string memory message) {
        require(hasRole(STAKEHOLDER_ROLE, msg.sender), message);
        _;
    }
```
```
    modifier onlyContributor(string memory message) {
        require(hasRole(CONTRIBUTOR_ROLE, msg.sender), message);
        _;
    }
```

These modifiers will be used to control access to specific functions.

```
    function createProposal(
        string calldata description,
        address charityAddress,
        uint256 amount
    )
        external
        onlyStakeholder("Only stakeholders are allowed to create proposals")
    {
        uint256 proposalId = numOfProposals++;
        CharityProposal storage proposal = charityProposals[proposalId];
        proposal.id = proposalId;
        proposal.proposer = payable(msg.sender);
        proposal.description = description;
        proposal.charityAddress = payable(charityAddress);
        proposal.amount = amount;
        proposal.livePeriod = block.timestamp + minimumVotingPeriod;
        emit NewCharityProposal(msg.sender, amount);
    }
```
`createProposal` is a function that interacts with blockchain state, which we can call in our dApp to add a new Proposal, it accepts three parameters: `description`, `charityAddress` and `amount`.

The function visibility is specified as external for economy of gas, which is also why the description will be contained in `calldata`. Memory allocation is expensive in terms of gas used, while reading from calldata is not. Also notice the `onlyStakeholder` modifier in use here, only accounts listed as Stakeholders can successfully call this function.

First a new identifier is set - the `proposalId`, which will be the existing number of proposals incremented by one. Next, we declare a proposal variable of the CharityProposal type using the storage keyword to make sure the state variable is maintained, then assign its reference to one of the buckets in the `charityProposals` mapping.
The remainder of the function assigns the necessary values to their counterparts in the CharityProposal object. payable() helps us by [converting the address literal](https://docs.soliditylang.org/en/v0.8.6/080-breaking-changes.html?highlight=payable#new-restrictions) into a payable address.

Finally, the `NewCharityProposal` event is emitted.

```
    function vote(uint256 proposalId, bool supportProposal)
        external
        onlyStakeholder("Only stakeholders are allowed to vote")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];


        votable(charityProposal);


        if (supportProposal) charityProposal.votesFor++;
        else charityProposal.votesAgainst++;


        stakeholderVotes[msg.sender].push(charityProposal.id);
    }
```
The `vote()` function is an external function that allows voting on proposals when called with the proposal’s id and a second true/false argument depending on whether the vote is in support or against the proposal.

```
    function votable(CharityProposal storage charityProposal) private {
        if (
            charityProposal.votingPassed ||
            charityProposal.livePeriod <= block.timestamp
        ) {
            charityProposal.votingPassed = true;
            revert("Voting period has passed on this proposal");
        }


        uint256[] memory tempVotes = stakeholderVotes[msg.sender];
        for (uint256 votes = 0; votes < tempVotes.length; votes++) {
            if (charityProposal.id == tempVotes[votes])
                revert("This stakeholder already voted on this proposal");
        }
    }
```

`votable()` is called within the `vote()` function. It is used to verify if a proposal can be voted on.

```
    function payCharity(uint256 proposalId)
        external
        onlyStakeholder("Only stakeholders are allowed to make payments")
    {
        CharityProposal storage charityProposal = charityProposals[proposalId];
        if (charityProposal.paid)
            revert("Payment has been made to this charity");
        if (charityProposal.votesFor <= charityProposal.votesAgainst)
            revert(
                "The proposal does not have the required amount of votes to pass"
            );
        charityProposal.paid = true;
        charityProposal.paidBy = msg.sender;
        emit PaymentTransfered(
            msg.sender,
            charityProposal.charityAddress,
            charityProposal.amount
        );
        return charityProposal.charityAddress.transfer(charityProposal.amount);
}
```

`payCharity` handles payment to the specified address after the voting period of the proposal has ended. It takes the proposalId as an argument and retreives that proposal from the mapping. We check whether the charity has already been paid or if the number of supporting votes is less than those against. If either of these conditions are true then the transaction will be reverted with an error message. If not, the paid property of the proposal is set true, the address of the stakeholder making the payment is set, and finally emit the PaymentTransfered event for logging purposes and transfer the payment to the charity address.

```
receive() external payable {
        emit ContributionReceived(msg.sender, msg.value);
}
```

This is needed so the contract can receive contributions without throwing an error. The function emits the ContributionReceived event previously declared in the upper scope.

```
function makeStakeholder(uint256 amount) external {
        address account = msg.sender;
        uint256 amountContributed = amount;
        if (!hasRole(STAKEHOLDER_ROLE, account)) {
            uint256 totalContributed =
                contributors[account] + amountContributed;
            if (totalContributed >= 5 ether) {
                stakeholders[account] = totalContributed;
                contributors[account] += amountContributed;
                _setupRole(STAKEHOLDER_ROLE, account);
                _setupRole(CONTRIBUTOR_ROLE, account);
            } else {
                contributors[account] += amountContributed;
                _setupRole(CONTRIBUTOR_ROLE, account);
            }
        } else {
            contributors[account] += amountContributed;
            stakeholders[account] += amountContributed;
        }
}
```

This function adds a new Stakeholder to the DAO if the total contribution of the user is more than or equal to 5 Celo. If the total contribution is less than 5 then the user is made a Contributor instead.

```
function getProposals()
        public
        view
        returns (CharityProposal[] memory props)
{
        props = new CharityProposal[](numOfProposals);


        for (uint256 index = 0; index < numOfProposals; index++) {
            props[index] = charityProposals[index];
        }
}
```

We are returning a list of all the proposals in the DAO here. Solidity doesn’t have iterators for the mapping type so we declare a fixed-size array, used the `numOfProposals` variable as the upper limit of our loop. For each iteration, we assign the proposal at the current index to the index in our fixed-size array then return the array. Essentially, this fetches our proposals and returns them as an array.

```
function getProposal(uint256 proposalId)
        public
        view
        returns (CharityProposal memory)
{
    return charityProposals[proposalId];
}
```

The `getProposal()` function takes a proposal id as an argument get the proposal from the proposal mapping then return the proposal.

```
function getStakeholderVotes()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256[] memory)
{
    return stakeholderVotes[msg.sender];
}
```

The `getStakeholderVotes()` gets and returns a list containing the id of all the proposals that a particular stakeholder has voted on.

```
function getStakeholderBalance()
        public
        view
        onlyStakeholder("User is not a stakeholder")
        returns (uint256)
{
    return stakeholders[msg.sender];
}
```

As the name suggests, the `getStakeholderBalance()` functions return the total amount of contribution a stakeholder has contributed to the DAO.

```
function isStakeholder() public view returns (bool) {
    return stakeholders[msg.sender] > 0;
}
```

The `isStakeholder()` function returns true/false depending on whether the caller is a stakeholder or not

```
function getContributorBalance()
        public
        view
        onlyContributor("User is not a contributor")
        returns (uint256)
{
    return contributors[msg.sender];
}
```

As the name suggests, this function returns the total balance of a contributor.

```
function isContributor() public view returns (bool) {
    return contributors[msg.sender] > 0;
}
```

The `isContributor()` function returns true/false depending on whether the caller is a contributor or not

&nbsp;

**Now that we have written the Smart Contract Code, the next step is to build the React Native App.**

[Create a React Native app](./create-a-react-native-app)

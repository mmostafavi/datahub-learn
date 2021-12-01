[**The original tutorial can be found in the Tezos documentation here**](https://assets.tqtezos.com/docs/oracle/). 

# Introduction

Smart contracts that require blockchain-external realtime data or events require a trusted _oracle_, i.e. a trusted smart contract that provides the data on-chain.

What is it useful for?

* An oracle that provides XTZ/USD prices could allow users to deposit funds that are immediately converted to some on-chain asset and represented as fiat.
* An oracle that provides weather data could allow a tsunami insurance contract: users purchase coverage from the contract, which pays out when there's a sufficiently severe tsunami in their area
* A combination of different oracles could be used to provide consensus, for example, where only results published by a majority of the oracles are considered valid.

# Setup

* Follow instructions in [Client Setup](https://assets.tqtezos.com/docs/setup/1-tezos-client) to set up `tezos-client` and create a test network wallet
* Clone [`lorentz-contract-oracle` repo](https://github.com/tqtezos/lorentz-contract-oracle) from GitHub and build from source using `stack`.

# The CLI

```text
❯❯❯ lorentz-contract-oracle Oracle --help
Usage: lorentz-contract-oracle Oracle COMMAND
  Oracle contract CLI interface

Available options:
  -h,--help                Show this help text

Available commands:
  print                    Dump the Oracle contract in form of Michelson code
  print-timestamped        Dump the Timestamped Oracle contract in form of
                           Michelson code
  init                     Initial storage for the Oracle contract
  get-value                get value
  update-value             update value
  update-admin             update admin
```

# Originating the contract

## Printing the contract

The print command takes a single argument: `valueType`, the type of the value provided by the oracle.

Note: Until [Issue \#6](https://github.com/tqtezos/lorentz-contract-oracle/issues/6) in the repo is fixed, the CLI tool will throw an error if `valueType` is _not_ `nat` when printing the contract or the `timestamped` version.

For example, if `nat`'s are provided:

```text
❯❯❯ lorentz-contract-oracle Oracle print --valueType "nat"

parameter (or (pair %getValue unit
                              (contract nat))
              (or (nat %updateValue)
                  (address %updateAdmin)));
storage (pair nat
              address);
code { CAST (pair (or (pair unit (contract nat)) (or nat address)) (pair nat address));
       DUP;
       CAR;
       DIP { CDR };
       IF_LEFT { DUP;
                 CAR;
                 DIP { CDR };
                 DIP { DIP { DUP };
                       SWAP };
                 PAIR;
                 CDR;
                 CAR;
                 DIP { AMOUNT };
                 TRANSFER_TOKENS;
                 NIL operation;
                 SWAP;
                 CONS;
                 PAIR }
               { IF_LEFT { DIP { DUP;
                                 CAR;
                                 DIP { CDR } };
                           DIP { DROP;
                                 DUP;
                                 DIP { SENDER;
                                       COMPARE;
                                       EQ;
                                       IF {  }
                                          { PUSH string "only admin may update";
                                            FAILWITH } } };
                           PAIR;
                           NIL operation;
                           PAIR }
                         { DIP { DUP;
                                 CAR;
                                 DIP { CDR };
                                 DIP { SENDER;
                                       COMPARE;
                                       EQ;
                                       IF {  }
                                          { PUSH string "only admin may update";
                                            FAILWITH } } };
                           SWAP;
                           PAIR;
                           NIL operation;
                           PAIR } } };
```

## Initial storage

```text
❯❯❯ lorentz-contract-oracle Oracle init --help
Usage: lorentz-contract-oracle Oracle init --initialValueType Michelson Type
                                           --initialValue Michelson Value
                                           --admin ADDRESS
  Initial storage for the Oracle contract

Available options:
  -h,--help                Show this help text
  --initialValueType Michelson Type
                           The Michelson Type of initialValue
  --initialValue Michelson Value
                           The Michelson Value: initialValue
  --admin ADDRESS          Address of the admin.
  -h,--help                Show this help text
```

Since we're using `nat`, `initialValueType` is `nat` and `initialValue` can be `0`:

```text
❯❯❯ lorentz-contract-oracle Oracle init \
  --initialValueType "nat" \
  --initialValue 3 \
  --admin $ALICE_ADDRESS
Pair 3 "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr"
```

## Running the origination

> **NOTE:** to use the annotated \(timestamped\) version, e.g. for `nat`:

```text
❯❯❯ tezos-client --wait none originate contract NatOracle \
  transferring 0 from $ALICE_ADDRESS running \
  "$(lorentz-contract-oracle Oracle print-timestamped --valueType "nat")" \
  --init "$(lorentz-contract-oracle Oracle init \
  --initialValueType "pair timestamp nat" --initialValue 'Pair "2019-12-10T17:43:53Z" 26871' \
  --admin $ALICE_ADDRESS)" --burn-cap 0.859 --force
```

> **NOTE:** the `nat_oracle.tz` contract may be found [here](https://github.com/tqtezos/lorentz-contract-oracle/blob/master/nat_oracle.tz)

Otherwise:

```text
❯❯❯ tezos-client --wait none originate contract NatOracle \
  transferring 0 from $ALICE_ADDRESS running \
  "$(lorentz-contract-oracle Oracle print --valueType "nat")" \
  --init "$(lorentz-contract-oracle Oracle init \
  --initialValueType "nat" --initialValue 3 \
  --admin $ALICE_ADDRESS)" --burn-cap 0.612

Waiting for the node to be bootstrapped before injection...
Current head: BLE8kx6VhXbk (timestamp: 2020-04-03T15:28:43-00:00, validation: 2020-04-03T15:29:02-00:00)
Node is bootstrapped, ready for injecting operations.
Estimated gas: 20483 units (will add 100 for safety)
Estimated storage: 683 bytes added (will add 20 for safety)
Operation successfully injected in the node.
Operation hash is 'onsEwbbm71wpzgnk6baBGacDzsnKceypEL841STJ6B5V9cA4swr'
NOT waiting for the operation to be included.
Use command
  tezos-client wait for onsEwbbm71wpzgnk6baBGacDzsnKceypEL841STJ6B5V9cA4swr to be included --confirmations 30 --branch BLE8kx6VhXbka6FGjXmKxnH56iypJLYZ9pM5mM3HLKvoVDBiMeS
and/or an external block explorer to make sure that it has been included.
This sequence of operations was run:
  Manager signed operations:
    From: tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm
    Fee to the baker: ꜩ0.002729
    Expected counter: 623928
    Gas limit: 20583
    Storage limit: 703 bytes
    Balance updates:
      tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm ............. -ꜩ0.002729
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,154) ... +ꜩ0.002729
    Origination:
      From: tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm
      Credit: ꜩ0
      Script:
        {...}
        Initial storage: (Pair 3 "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr")
        No delegate for this contract
        This origination was successfully applied
        Originated contracts:
          KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp
        Storage size: 426 bytes
        Paid storage size diff: 426 bytes
        Consumed gas: 20483
        Balance updates:
          tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm ... -ꜩ0.426
          tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm ... -ꜩ0.257

New contract KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp originated.
Contract memorized as NatOracle.
```

Make an alias for the resulting address:

```text
❯❯❯ ORACLE_ADDRESS="KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp"
```

# Getting a value

## Preparing a view contract

Originate the contract:

```text
❯❯❯ tezos-client --wait none originate contract nat_storage transferring 0 \
  from $ALICE_ADDRESS running "$(lorentz-contract print --name NatStorageContract)" \
  --init 0 --burn-cap 0.295
```

Make an alias for its address:

```text
❯❯❯ NAT_STORAGE_ADDRESS="KT1ShW17HERZgjUTxSxSc4W3tuWrfLCqBmEi"
```

See the [`FA1.2` Quickstart](https://assets.tqtezos.com/docs/token-contracts/fa12/3-fa12-lorentz) for more info.

## Make the parameter

```text
❯❯❯ lorentz-contract-oracle Oracle get-value --help
Usage: lorentz-contract-oracle Oracle get-value --callbackContract ADDRESS
  get value

Available options:
  -h,--help                Show this help text
  --callbackContract ADDRESS
                           Address of the callbackContract.
  -h,--help                Show this help text
```

```text
❯❯❯ lorentz-contract-oracle Oracle get-value --callbackContract $NAT_STORAGE_ADDRESS
Left (Pair Unit "KT1ShW17HERZgjUTxSxSc4W3tuWrfLCqBmEi")
```

## Get the value

```text
❯❯❯ tezos-client --wait none transfer 0 from $ALICE_ADDRESS to $ORACLE_ADDRESS \
  --arg "$(lorentz-contract-oracle Oracle get-value \
  --callbackContract $NAT_STORAGE_ADDRESS)" --burn-cap 0.000001

Waiting for the node to be bootstrapped before injection...
Current head: BL7fbZE3Gs2V (timestamp: 2020-04-03T15:33:25-00:00, validation: 2020-04-03T15:33:45-00:00)
Node is bootstrapped, ready for injecting operations.
Estimated gas: 31566 units (will add 100 for safety)
Estimated storage: no bytes added
Operation successfully injected in the node.
Operation hash is 'opWqWvkJczWzq5m99svSDzW7HmWY5nCUB1rerDn6utLLH5NJ1dS'
NOT waiting for the operation to be included.
Use command
  tezos-client wait for opWqWvkJczWzq5m99svSDzW7HmWY5nCUB1rerDn6utLLH5NJ1dS to be included --confirmations 30 --branch BL7fbZE3Gs2VkmziY11S4biX9cfuLVnQmLr8r7ALbJukxD4tRSY
and/or an external block explorer to make sure that it has been included.
This sequence of operations was run:
  Manager signed operations:
    From: tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm
    Fee to the baker: ꜩ0.00347
    Expected counter: 623930
    Gas limit: 31666
    Storage limit: 0 bytes
    Balance updates:
      tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm ............. -ꜩ0.00347
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,154) ... +ꜩ0.00347
    Transaction:
      Amount: ꜩ0
      From: tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm
      To: KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp
      Parameter: (Left (Pair Unit "KT1ShW17HERZgjUTxSxSc4W3tuWrfLCqBmEi"))
      This transaction was successfully applied
      Updated storage:
        (Pair 3 0x00003b5d4596c032347b72fb51f688c45200d0cb50db)
      Storage size: 426 bytes
      Consumed gas: 20242
    Internal operations:
      Transaction:
        Amount: ꜩ0
        From: KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp
        To: KT1ShW17HERZgjUTxSxSc4W3tuWrfLCqBmEi
        Parameter: 3
        This transaction was successfully applied
        Updated storage: 3
        Storage size: 38 bytes
        Consumed gas: 11324
```

# Update the value

## Make the parameter

```javascript
❯❯❯ lorentz-contract-oracle Oracle update-value --help
Usage: lorentz-contract-oracle Oracle update-value --newValueType Michelson Type
                                                   --newValue Michelson Value
  update value

Available options:
  -h,--help                Show this help text
  --newValueType Michelson Type
                           The Michelson Type of newValue
  --newValue Michelson Value
                           The Michelson Value: newValue
  -h,--help                Show this help text
```

## Update the value

To update the value to `4`:

```text
Waiting for the node to be bootstrapped before injection...
Current head: BMLjQ5pgcmxd (timestamp: 2020-04-03T15:35:39-00:00, validation: 2020-04-03T15:35:44-00:00)
Node is bootstrapped, ready for injecting operations.
Estimated gas: 19463 units (will add 100 for safety)
Estimated storage: no bytes added
Operation successfully injected in the node.
Operation hash is 'ooEf7sgpvhMcUBcn66iMLaWNzs4TEAxb4icjXKF27xM2QPMTGQP'
NOT waiting for the operation to be included.
Use command
  tezos-client wait for ooEf7sgpvhMcUBcn66iMLaWNzs4TEAxb4icjXKF27xM2QPMTGQP to be included --confirmations 30 --branch BMLjQ5pgcmxdN11kfhA2NS4igfSdJeL5HbopDn8LyAckBroWZhi
and/or an external block explorer to make sure that it has been included.
This sequence of operations was run:
  Manager signed operations:
    From: tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr
    Fee to the baker: ꜩ0.001259
    Expected counter: 632294
    Gas limit: 10000
    Storage limit: 0 bytes
    Balance updates:
      tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr ............. -ꜩ0.001259
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,154) ... +ꜩ0.001259
    Revelation of manager public key:
      Contract: tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr
      Key: edpkvCHgVArnZo9RTP4P6euLTyhE89u73CYjBgsP4wEJbj4quao9oR
      This revelation was successfully applied
      Consumed gas: 10000
  Manager signed operations:
    From: tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr
    Fee to the baker: ꜩ0.002123
    Expected counter: 632295
    Gas limit: 19563
    Storage limit: 0 bytes
    Balance updates:
      tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr ............. -ꜩ0.002123
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,154) ... +ꜩ0.002123
    Transaction:
      Amount: ꜩ0
      From: tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr
      To: KT1LVLUsC1WCo4yHzeoDEE8FxxoC7EW6bSyp
      Parameter: (Right (Left 4))
      This transaction was successfully applied
      Updated storage:
        (Pair 4 0x00003b5d4596c032347b72fb51f688c45200d0cb50db)
      Storage size: 426 bytes
      Consumed gas: 19463
```

# Setting up a server

While we have everything we need to originate and administer an oracle contract, we'd like to automatically push updates to the contract.

Below are two simple examples of how you can periodically fetch some data and automatically push it to the oracle contract.

## Minimal server implementation

A minimal server written in bash shellscript could consist of:

* A Bash script `update_value.sh` that gets the `NEW_VALUE` and then calls the `update-value` entrypoint:

```bash
##!/bin/bash
## update_value.sh

NEW_VALUE="$(my_get_new_value_script)

tezos-client --wait none transfer 0 from $ALICE_ADDRESS to $ORACLE_ADDRESS \
  --arg "$(lorentz-contract-oracle Oracle update-value \
  --newValueType "nat" --newValue $NEW_VALUE)" --burn-cap 0.000001
```

* A `crontab` job to run `update_value.sh` periodically \(in the example it's daily at `5:03 pm`\):

```text
03 05 * * * update_value.sh
```

## Docker Flask server

There is an implementation of the above Bash server in Python, packaged as a Docker image, which:

* Fetches the latest stock price from Alpha Vantage
* Runs a "cron job" to push the update to the oracle contract with [`pytezos`](https://github.com/baking-bad/pytezos) every 30 seconds
* Serves a webpage for debugging

It does the same thing as the Bash script where `my_get_new_value_script` fetches the latest ticker price from Alpha Vantage.

You can find the project on Github [here](https://github.com/tqtezos/lorentz-contract-oracle/tree/master/stock_ticker) or view the one-file Python code [here](https://github.com/tqtezos/lorentz-contract-oracle/blob/master/stock_ticker/tq/oracles/ticker.py).

To get the image from [the DockerHub repo](https://hub.docker.com/r/tqtezos/oracle-stock-ticker), run:

```text
❯❯❯ docker pull tqtezos/oracle-stock-ticker:2.1
```

The server is configured using environment variables:

* To generate the `TEZOS_USER_KEY` parameter, run: `echo "$(base64 MY_KEY_FILE.json | tr -d '\n')"`, where `MY_KEY_FILE.json` is your Tezos faucet file \(see [here](https://faucet.tzalpha.net/) to get a testnet faucet file\).
* You can get a free `ALPHA_VANTAGE_API_KEY` from [Alpha Vantage](https://www.alphavantage.co/support/#api-key)
* Alpha Vantage's `Search Endpoint` may be used to find `ALPHA_VANTAGE_TICKER_SYMBOL`'s

```
TEZOS_USER_KEY=".."
ORACLE_ADDRESS="KT1CUTjTqf4UMf6c9A8ZA4e1ntWbLVTnvrKG"
ALPHA_VANTAGE_API_KEY=".."
ALPHA_VANTAGE_TICKER_SYMBOL="AAPL"
FLASK_APP="tq/oracles/ticker.py"
```

To run the Docker container:

```javascript
❯❯❯ docker run -d -p 5000:5000 \
  --env TEZOS_USER_KEY=".."
  --env ORACLE_ADDRESS="KT1.." \
  --env ALPHA_VANTAGE_API_KEY=".." \
  --env ALPHA_VANTAGE_TICKER_SYMBOL="AAPL" \
  --env FLASK_APP="tq/oracles/ticker.py" \
  oracle-stock-ticker
```

Once it's up, it will serve the debug page and update the oracle contract roughly every 30 seconds.

You should be able to view the debug screen at `localhost:5000`, fetch the latest values using `get-value`, and use a block explorer to inspect the contract.

You can find a running example of the contract [on Carthage](https://you.better-call.dev/carthage/KT1CUTjTqf4UMf6c9A8ZA4e1ntWbLVTnvrKG/operations).

If you had any difficulties following this tutorial or simply want to discuss Tezos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

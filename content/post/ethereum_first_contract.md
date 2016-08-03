+++
date = "2016-08-04T13:00:00+08:00"
title = "Ethereum first smart contract"
description = "Ethereum dev environment"
tags = [ "golang", "blockchain", "ethereum", "smart contract" ]
topics = [ "ethereum", "smart contract" ]
slug = "ethereum-first-contract"
+++

## Launch your geth (or testrpc) private instance

```
./geth \
    --identity "gethTest" \
    --rpc --rpcport "9012" \
    --rpccorsdomain "YOUR_TEST_DOMAIN_APP_RUN_FROM" \
    --datadir "./testChain" \
    --port "30303" \
    --nodiscover \
    --rpcapi "db,eth,net,web3" \
    --networkid 1999 \
    --dev console
```

## Within the console, compile your contract

check this tutorial: https://www.ethereum.org/greeter

I had an issue when I followed the contract tutorial, my contract would not be mined after I was trying to deploy it.

The issue was that my account was locked :/ and the greeter contract stupidly silently fails...
Here is the modified code to see the obvious error.

```
var _greeting = "Hello World!"
var greeterContract = web3.eth.contract(greeterCompiled.greeter.info.abiDefinition);

var greeter = greeterContract.new(_greeting,{from:web3.eth.accounts[0], data: greeterCompiled.greeter.code, gas: 30000000}, function(e, contract){
    if(!e) {

      if(!contract.address) {
        console.log("Contract transaction send: TransactionHash: " + contract.transactionHash + " waiting to be mined...");

      } else {
        console.log("Contract mined! Address: " + contract.address);
        console.log(contract);
      }

    } else {
        console.log("error!");
        console.log(e);
    }
})
```

which returns

```
error!
Error: account is locked
```

To unlock the account, simply:

```
personal.unlockAccount(eth.accounts[0])
```

I then encountered ANOTHER error!

```
error!
Error: Exceeds block gas limit
```

I simply decreased the gas to 3000000, and finally got:

```
I0803 15:14:29.423492 eth/api.go:1191] Tx(0x7584a963a9c2a21e623e607826ad47ae358d056ed159b82a21793d4541148e86) created: 0x5f3425ccedeae0eb36521c4cf93ec6544dbad9bd
Contract transaction send: TransactionHash: 0x7284a963a9c3a21e623e607826ad47ae358d056ed159b82a21793d4541148e86 waiting to be mined...
```

Then, simply mine it!

```
miner.start()
...
Contract mined! Address: 0x5d3125ccedeae0eb36521c4cf93ec6544dbad9bd
...
miner.stop()
```

Yeah, victory! my contract is deployed on my private ethereum node.

Next we will simply interact with the contract.
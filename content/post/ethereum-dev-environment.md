+++
date = "2016-08-01T08:27:27+08:00"
title = "Ethereum dev environment"
description = "Ethereum dev environment"
tags = [ "golang", "blockchain", "ethereum", "geth", "dev", "testrpc" ]
topics = [ "dev", "ethereum", "testrpc" ]
slug = "ethereum-dev-environment"
+++

## Part 1 - Setup your ethereum node

There are many ways you can setup a node to dev an Ethereum dapp.
You can use the live network: not advisable obviously for cost and speed reasons.
You can use the test network: not advisable for speed reasons.
You can use a testchain set up with Geth: easy but a bit tedious as you need to mine.
You can the ethereum testrpc: easiest!

I will talk about the last two setup in this article.

### Using testrpc

Simply install through npm (if you want it globally available, add -g after install, as usual)
```
npm install ethereumjs-testrpc
```

And then run it
```
node_modules/ethereumjs-testrpc/bin/testrpc
```

### Using geth

Download geth latest release (https://github.com/ethereum/go-ethereum/releases)
and extract it.

Create a file customGenesis.json
```json
{
  "nonce": "0x0000000000000042",
  "timestamp": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "0x0",
  "gasLimit": "0x8000000",
  "difficulty": "0x400",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x3333333333333333333333333333333333333333",
  "alloc": {}
}
```

then init yout node with the genesis block above
```shell
chmod +x geth
./geth init ./customGenesis.json
```

#### Run your node with console attached

```shell
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

#### Create a base account

```
> eth.accounts
[]
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0xedea6958c57fc0cd4bd63b3e7b395393dc76bfb6"
> eth.accounts
["0xedea6958c57fc0cd4bd63b3e7b395393dc76bfb6"]
```

#### Mine on your newly created account

```
miner.setEtherbase(eth.accounts[0])
miner.start(8)
miner.stop()
```

Check if the mining worked

```
> eth.getBalance(eth.accounts[0]).toNumber();
55000000000000000000
```

In the next posts, we will start talking about development of dapps.
+++
date = "2015-08-20T08:27:27+08:00"
title = "Ethereum dev environment"
description = "Ethereum dev environment"
tags = [ "golang", "blockchain", "ethereum", "geth", "dev", "testrpc" ]
topics = [ "dev", "ethereum", "testrpc" ]
slug = "ethereum-dev-environment"
+++

# Part 1 - Setup your ethereum node

There are many ways you can setup a node to dev an Ethereum dapp.
You can use the live network: not advisable obviously for cost and speed reasons.
You can use the test network: not advisable for speed reasons.
You can use a testchain set up with Geth: easy but a bit tedious as you need to mine.
You can the ethereum testrpc: easiest!

I will talk about the last two setup in this article

## Using geth

go here [http://ethdocs.org/en/latest/network/test-networks.html#id3]

the --genesis flag is deprecated. Just use:

```
./geth init ./customGenesis.json
```

### Mine on your newly created account

```
miner.setEtherbase(eth.accounts[0])
miner.start(8)
miner.stop()
```

## Using testrpc

Simply install through npm (if you want it globally available, add -g after install, as usual)
```
npm install ethereumjs-testrpc
```

And then run it
```
node_modules/ethereumjs-testrpc/bin/testrpc
```

In the next posts, we will start talking about development of dapps.
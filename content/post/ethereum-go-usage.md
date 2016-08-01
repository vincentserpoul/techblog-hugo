+++
date = "2015-08-20T08:27:27+08:00"
title = "Ethreum and golang"
description = "Ethereum Golang"
tags = [ "golang", "blockchain", "ethereum" ]
topics = [ "golang", "ethereum" ]
slug = "ethereum-go"
+++

### Setup a local private tests

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

You re all set!

### Dapp dev

Go check truffle [http://truffle.readthedocs.io/en/latest/getting_started/project/]

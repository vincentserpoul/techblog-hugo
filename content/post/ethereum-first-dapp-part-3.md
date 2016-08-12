+++
date = "2016-08-12T12:00:27+08:00"
title = "Ethereum first dapp - part 3"
description = "Ethereum first dapp - part 3"
tags = [ "ethereum", "dapp", "development", "metamask" ]
topics = [ "dev", "ethereum", "metamask" ]
slug = "ethereum-first-dapp-part-3"
+++

## setting up a light wallet
In order to have a setup close to what the DAPP would be, we will use (metamask) [http://www.metamask.io] as a light wallet (there are other choices).

Metamask allows you to connect to a custom node.

We will then connect to our node, http://localhost:9012
If everthing is fine, metamasks should indicate it's connected.

Then, we can import the metamask account to our local node by simply specifying the datadir we have setup the node data.

Let's imagine metamask give us:

```
0xcddddd76a763de6e9ac19e7ec8f71ed77d61eebc0a93068fc9f2a544462db7d2
```

as the key.

Then let's just copy this, without thw 0x inside a separate file.

```shell
echo "cddddd76a763de6e9ac19e7ec8f71ed77d61eebc0a93068fc9f2a544462db7d2" >> pvkey.txt
```

and then import it in our local chain

```
geth --datadir "./testChain" account import ./pvkey.txt
```

if you look at the list of accounts in your local node, you should see now the newly imported account.

## give me ether!

Now, simply transfer some Ether to the newly imported account
In geth, for example:

```
eth.sendTransaction({from: '0xfffffff', to: '0xddddddd', value: web3.toWei(1000, "ether")})
```

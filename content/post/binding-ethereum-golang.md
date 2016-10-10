+++
date = "2016-10-10T12:54:00+08:00"
title = "Ethereum contracts and Golang"
description = "binding Ethereum contracts in golang"
tags = [ "golang", "blockchain", "ethereum", "dev", "testrpc" ]
topics = [ "dev", "ethereum", "testrpc" ]
slug = "binding-ethereum-golang"
+++

## The contract

```solidity
contract Trigger {
  function () {
      throw;
  }

  address owner;

  function Trigger() {
      owner = msg.sender;
  }

  event TriggerEvt(address _sender, uint _trigger);

  function trigger(uint _trigger) {
      TriggerEvt(msg.sender, _trigger);
  }

  function getOwner() constant returns (address) {
    return owner;
  }

}
```

This is a very simple contract that we will take as an example.

## Getting the right tools for binding

A good starting point is this [wiki](https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts).

You will need to follow the install procedure of [go-ethereum](https://github.com/ethereum/go-ethereum).

Once done, you should have the abigen executable available on your command line.

## Automatically generating the go file

```shell
abigen --sol contracts/Trigger.sol --pkg main --out trigger.go
```

If everything is fine, you should now have a file named trigger.go

## Using the generated file from main

You first need to have a node running (parity, geth, testrpc...). We will assume it's listening on port 9012.
You then need to deploy your contract and write down the deployment address (you can use truffle or simple deploy your contract manually or use the following code with a working key pair).

```golang
package main

import (
  "fmt"
  "log"
  "strings"
  "time"

  "github.com/ethereum/go-ethereum/accounts/abi/bind"
  "github.com/ethereum/go-ethereum/accounts/abi/bind/backends"
  "github.com/ethereum/go-ethereum/rpc"
)

func main() {
  // Create an IPC based RPC connection to a remote node
  conn, err := rpc.NewHTTPClient("http://localhost:9012")
  if err != nil {
    log.Fatalf("Failed to connect to the Ethereum client: %v", err)
  }

  // IF YOU WANT TO DEPLOY YOURSELF
  // this is the json found in your geth chain/keystore folder
  key := `{"address":"f2759b4a699dae4fdc3383a0d7a92cfc246315cd","crypto":{"cipher":"aes-128-ctr","ciphertext":"a96fe235356c7ebe6520d2fa1dcc0fd67199cb490fb18c39ffabbb6880a6b3d6","cipherparams":{"iv":"47182104a4811f8da09c0bafc3743e2a"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"81c82f97edb0ee1036e63d1de57b7851271273971803e60a5cbb011e85baa251"},"mac":"09f107c9af8efcb932354d939beb7b2c0cebcfd70362d68905de554304a7cfff"},"id":"eb7ed04f-e996-4bda-893b-28dc6ac24626","version":3}`
  auth, err := bind.NewTransactor(strings.NewReader(key), "1234567890")
  if err != nil {
    log.Fatalf("Failed to create authorized transactor: %v", err)
  }
  // Deploy a new awesome contract for the binding demo
  triggerAddr, _, trigger, err := DeployTrigger(auth, backends.NewRPCBackend(conn))
  if err != nil {
    log.Fatalf("Failed to deploy new trigger contract: %v", err)
  }
  // Don't even wait, check its presence in the local pending state
  time.Sleep(5 * time.Second) // Allow it to be processed by the local node :P
  // END IF YOU WANT TO DEPLOY YOURSELF

  // IF YOU HAVE ALREADY DEPLOYED IT
  // deployedTriggerAddr := "0xe2359b4a699dae4fdc3383a0d7a92cfc246315ce"
  deployedTriggerAddr := triggerAddr
  trigger, err = NewTrigger(deployedTriggerAddr, backends.NewRPCBackend(conn))
  if err != nil {
    log.Fatalf("Failed to instantiate a trigger contract: %v", err)
  }
  // END IF YOU HAVE ALREADY DEPLOYED IT

  owner, err := trigger.GetOwner(nil)
  if err != nil {
    log.Fatalf("Failed to retrieve token name: %v", err)
  }
  fmt.Printf("owner address: 0x%x\n", owner)
}
```

then, just run it

```
go run *.go
owner address: 0xf2759b4a699dae4fdc3383a0d7a92cfc246315cd
```

Et voila!

## Existing issues

We have still not talked about listening to events.
There are also still issues as soon as the contract imports other contracts, I will finish the writing once these are done.
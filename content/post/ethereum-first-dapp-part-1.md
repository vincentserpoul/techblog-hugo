+++
date = "2016-08-08T15:00:27+08:00"
title = "Ethereum first dapp - part 1"
description = "Ethereum first dapp - part 1"
tags = [ "ethereum", "dapp", "development" ]
topics = [ "dev", "ethereum", "testrpc" ]
slug = "ethereum-first-dapp-part-1"
+++

# Contracts

## Prepare your folder for your dapp

```
mkdir dapp
```

inside this folder, we'll create one folder for truffle, one for geth.

```
cd dapp
mkdir truffle geth
```

inside the geth folder, simply put the customGenesis block you can find in the ethereum-dev-environment blog post.

We are going to use two Ethereum clients, one for tests and devs, testrpc and one for a more real interaction, geth.

Let's install truffle and testrpc

```
npm install ethereumjs-testrpc truffle
```

## Truffle

Truffle is a-m-a-z-i-n-g for contract development. It will allow you to unit test the contracts and compile them into usable javascript objects usable in web3! Just what we needed!

First, init truffle scaffolding (I don't like -g install XD, so bear with my node_modules folder).

```
node_modules/truffle/cli.js init
```

This will create different folders. You can have a look at http://truffle.readthedocs.io/ to have more details.

## Testrpc

Let's launch the ethereumjs-testrpc

```
node_modules/ethereumjs-testrpc/bin/testrpc
```

You will see it will create a new chain, in memory and create 10 new accounts for you to test.

## Standard token contracts

Don't try to reinvent the wheel, Consensys has given standard contracts to issue your own token.
Have a look here: https://github.com/ConsenSys/Tokens

Just copy to your contract folder:

* Tokens/Token_Contracts/contracts/HumanStandardToken.sol
* Tokens/Token_Contracts/contracts/StandardToken.sol
* Tokens/Token_Contracts/contracts/Token.sol

## Migrate

Here is the simple migration script I used

```
module.exports = (deployer) => {
  deployer.deploy(
    HumanStandardToken,
        1000000,
        'mold',
        '0',
        'M'
  );
};
```

In order to compile and deploy your contracts in the rpc node you configured in truffle.js, you need to launch the following command:

```
node_modules/truffle/cli.js migrate
```

If your code compiles, it will create javascript objects for each of your contract in the folder build/contracts.

## Testing

Here is an example of tests on HumanStandardToken (ES6)

```
const it = require('mocha').it;
const assert = require('chai').assert;

contract('HumanStandardToken', (accounts) => {
  it('should put 1000000 molds in the first account', () => {
    const molds = HumanStandardToken.deployed();
    const initSupply = 1000000;

    return molds.balanceOf(accounts[0])
      .then(balance => assert.equal(
          balance.valueOf(), initSupply,
          `${initSupply} molds weren't in the first account`
        )
      );
  });

  it('should transfer molds to another user', () => {
    const molds = HumanStandardToken.deployed();

    return molds.transfer(accounts[1], 100, { from: accounts[0] })
      .then(
        () => molds.balanceOf(accounts[1])
      )
      .then(balance => assert.equal(
          balance.valueOf(), 100,
          `${accounts[1]} didn't was not transfered 100 molds but ${balance.valueOf()}`
        )
      );
  });
});
```

** / ! \ **
I was stuck for hours because of the way contract functions can be called.
There are two types of actions on the Ethereum blockchain. Action which change the contract states (like transfer funds for example), and the one which don't (like get balance, for example).
In truffle, you call the latter with a "call" on the contract object, whereas the former doesn't need it... Confusing, isn't it?
I found out later that there are constant functions in solidity, which basically mean functions that don't change the contract state. When you specify these functions in you contract definition as "constant", then, truffle doesn't need the call... Problem solved, everything now looks uniform.

Now you can try if everything works fine, launch testrpc

```
node_modules/ethereumjs-testrpc/bin/testrpc
```

and open a new terminal and launch the tests

```
node_modules/truffle/cli.js test
```

## Creation of the contracts on a private geth or even morden

Modify your truffle.js to whichever node you want to migrate your contracts to.
Mine is local geth, listening on port 9012, so truffle.js will become

```
module.exports = {
  rpc: {
    host: 'localhost',
    port: 9012,
  },
};
```

then migrate

```
node_modules/truffle/cli.js migrate
```

Et voila! if everything is ok, truffle should have created your contracts and updated the built contracts with the right contract addresses in build/contracts.

Now, let's play with these contracts on geth! See you on part 2.

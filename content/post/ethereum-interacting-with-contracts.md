+++
date = "2016-08-03T20:00:00+08:00"
title = "Interacting with an Ethereum smart contracts"
description = "frontend javascript to interact with smart contracts"
tags = [ "blockchain", "ethereum", "smart contract", "javascript", "web3", "dapp" ]
topics = [ "ethereum", "smart contract", "javascript", "web3", "dapp" ]
slug = "ethereum-interacting-with-contracts"
+++

## Check the address of the current deployed contract

Remember when you mined your contract, it told your its address.
Now, reuse it!

```
eth.getCode("0x5f3425ccedeae0eb36521c4cf93ec6544dbad9bd")
```

## Test the contract with a simple interaction

get the latest web3-light.min.js js from https://github.com/ethereum/web3.js/releases and simply copy the dist/web3-light.min.js into the same folder as the following HTML file.

then, use this html to interact with your contract on the local node:

```
<!doctype>
<html>
<head>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/bignumber.js/2.4.0/bignumber.min.js"></script>
    <script type="text/javascript" src="./web3-light.min.js"></script>
    <script type="text/javascript">
    var Web3 = require('web3');
    var web3 = new Web3();
    web3.setProvider(new web3.providers.HttpProvider('http://localhost:9012'));
    function watchBalance() {
        var coinbase = web3.eth.coinbase;
        var originalBalance = web3.eth.getBalance(coinbase).toNumber();
        document.getElementById('coinbase').innerText = 'coinbase: ' + coinbase;
        document.getElementById('original').innerText = ' original balance: ' + originalBalance + '    watching...';
        web3.eth.filter('latest').watch(function() {
            var currentBalance = web3.eth.getBalance(coinbase).toNumber();
            document.getElementById("current").innerText = 'current: ' + currentBalance;
            document.getElementById("diff").innerText = 'diff:    ' + (currentBalance - originalBalance);
        });
    }
    </script>
</head>
<body>
    <h1>coinbase balance</h1>
    <button type="button" onClick="watchBalance();">watch balance</button>
    <div></div>
    <div id="coinbase"></div>
    <div id="original"></div>
    <div id="current"></div>
    <div id="diff"></div>
</body>
</html>

```

and... It works, you can see the balance on the contract!

## Let's greet now

```
<!doctype>
<html>
<head>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/bignumber.js/2.4.0/bignumber.min.js"></script>
    <script type="text/javascript" src="./web3-light.min.js"></script>
    <script type="text/javascript">
        var Web3 = require('web3');
        var web3 = new Web3();
        web3.setProvider(new web3.providers.HttpProvider('http://localhost:9012'));
        function watchBalance() {
            var coinbase = web3.eth.coinbase;
            var originalBalance = web3.eth.getBalance(coinbase).toNumber();
            document.getElementById('coinbase').innerText = 'coinbase: ' + coinbase;
            document.getElementById('original').innerText = ' original balance: ' + originalBalance + '    watching...';
            web3.eth.filter('latest').watch(function() {
                var currentBalance = web3.eth.getBalance(coinbase).toNumber();
                document.getElementById("current").innerText = 'current: ' + currentBalance;
                document.getElementById("diff").innerText = 'diff:    ' + (currentBalance - originalBalance);
            });
        }
        function greet() {
            var contractAddress = '0x5d3425ccedeae0eb36521c4cf93ec6544dbad9bd';
            var greeter = web3.eth.contract([{constant:false,inputs:[],name:'kill',outputs:[],type:'function'},{constant:true,inputs:[],name:'greet',outputs:[{name:'',type:'string'}],type:'function'},{inputs:[{name:'_greeting',type:'string'}],type:'constructor'}]).at(contractAddress);
            var greetings = greeter.greet();
            document.getElementById('greeting').innerText = greetings;
        }
    </script>
</head>
<body>
    <h1>coinbase balance</h1>
    <button type="button" onClick="watchBalance();">watch balance</button>
    <div></div>
    <div id="coinbase"></div>
    <div id="original"></div>
    <div id="current"></div>
    <div id="diff"></div>
    <h1>greetings</h1>
    <button type="button" onClick="greet();">greet!</button>
    <div id="greeting"></div>
</body>
</html>
```

And you have it! the dapp is responding!

caveat: when you won't be running a test, you will need to get an http provider connected to the live blockchain. You can be sure to have one if you run your own node, I'm not sure yet if there is any open htt provider node out there.
+++
date = "2016-10-27T13:34:00+08:00"
title = "Ethereum react dapps"
description = "Ethereum contracts and dapps"
tags = [ "react", "blockchain", "ethereum", "dev", "dapps" ]
topics = [ "dev", "ethereum", "testrpc", "react" ]
slug = "ethereum-react-dapps"
+++

I finished my first dapp with (react-boilerplate)[https://github.com/mxstbr/] this week and here are the few things I learnt.
I won't get into the redux, redux-saga details, I let you play with the amazing boilerplate.

## Interacting with constant functions

Let's use the typical balanceOf function of the EIP20 contracts:

```solidity
    function balanceOf(address _owner) constant returns (uint256 balance) {
        return balances[_owner];
    }
```

Here are the sagas (redux-sagas) I used to interact:

```javascript
import { take, call, put, cancel, select, fork } from 'redux-saga/effects';
import {
  BALANCE_OF_GET,
} from './constants';
import { LOCATION_CHANGE } from 'react-router-redux';

import {
  balanceOfSuccess,
  balanceOfFailure,
} from './actions';

import { selectEthConnectWeb3Connection } from 'containers/EthConnect/selectors';

import HumanStandardToken from 'contracts/HumanStandardToken.sol.js';

function* balanceOfGet(ethAddress) {
  const web3Connection = yield select(selectEthConnectWeb3Connection());

  HumanStandardToken.setProvider(web3Connection.currentProvider);
  const token = HumanStandardToken.deployed();

  const balancePromise = yield call(token.balanceOf, ethAddress);

  // We return an object in a specific format, see utils/request.js for more information
  if (balancePromise.err === undefined || balancePromise.err === null) {
    yield put(balanceOfSuccess(ethAddress, balancePromise.valueOf()));
  } else {
    console.log(balancePromise.err.response); // eslint-disable-line no-console
    yield put(balanceOfFailure(ethAddress, balancePromise.err.response));
  }
}

/**
 * Watches for BALANCE_OF_GET action and calls handler
 */
export function* balanceOfWatcher() {
  while (true) { // eslint-disable-line no-constant-condition
    const { ethAddress } = yield take(BALANCE_OF_GET);
    // use fork and not call to be sure to fork all
    yield fork(balanceOfGet, ethAddress);
  }
}


/**
 * Root saga manages watcher lifecycle
 */
export function* balanceOfSaga() {
  // Fork watcher so we can continue execution
  const watcher = yield fork(balanceOfWatcher);

  // Suspend execution until location changes
  yield take(LOCATION_CHANGE);
  yield cancel(watcher);
}

// All sagas to be loaded
export default [
  balanceOfSaga,
];

```

Then, simply implement the right reducers, actions and you are done! you can easily get the balance in your UI.

## Interacting with functions that needs transactions

Let's use the same EIP20 contract again:

```solidity
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);

    function approve(address _spender, uint256 _value) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
        return true;
    }
```

```javascript
import { take, call, put, select, cancel, fork } from 'redux-saga/effects';
import {
  GIVEMETOKEN_LAUNCH,
} from './constants';
import { LOCATION_CHANGE } from 'react-router-redux';

import {
  givemetokensSuccess,
  givemetokensFailure,
} from './actions';

import {
  balanceOfGet,
} from 'containers/Token/actions';

import { selectEthConnectWeb3Connection } from 'containers/EthConnect/selectors';
// import { selectRianEthAddress } from './selectors';

import HumanStandardToken from 'contracts/HumanStandardToken.sol.js';

function* givemetokens(ethAddress) {
  const web3Connection = yield select(selectEthConnectWeb3Connection());
  HumanStandardToken.setProvider(web3Connection.currentProvider);
  const tokens = HumanStandardToken.deployed();

  const tokensOwner = yield call(tokens.getOwner.call);

  try {
    yield call(tokens.approve, ethAddress, 100, { from: tokensOwner, gas: 200000 });
  } catch (e) {
    console.log(e); // eslint-disable-line no-console
    yield put(givemetokensFailure(ethAddress));
    return;
  }

  const approvedAmt = yield call(tokens.allowance.call, tokensOwner, ethAddress);
  console.log(`approved amount: ${approvedAmt}`);

  try {
    yield call(tokens.transferFrom, tokensOwner, ethAddress, 100, { from: ethAddress, gas: 200000 });
  } catch (e) {
    console.log(e); // eslint-disable-line no-console
    yield put(givemetokensFailure(ethAddress));
  }

  yield put(givemetokensSuccess(ethAddress, true));
  yield put(balanceOfGet(ethAddress));
}

/**
 * Watches for SIMPLEINSURANCE_givemetokens_LAUNCH action and calls handler
 */
export function* givemetokensWatcher() {
  while (true) { // eslint-disable-line no-constant-condition
    const { ethAddress } = yield take(GIVEMETOKEN_LAUNCH);
    yield call(givemetokens, ethAddress);
  }
}


/**
 * Root saga manages watcher lifecycle
 */
export function* givemetokensSaga() {
  // Fork watcher so we can continue execution
  const watcher = yield fork(givemetokensWatcher);

  // Suspend execution until location changes
  yield take(LOCATION_CHANGE);
  yield cancel(watcher);
}

// All sagas to be loaded
export default [
  givemetokensSaga,
];
```

At that point, you only know that the transaction has gone through, you don't really know what happened. That's when you need to listen to the event of your contracts.

## Listening to events

Same example as above, we will listen to the Approval event.
We need to create a eventChannel, as the event is coming from *outside* this time, it won't be generated by our own UI.

```javascript
const ethEvent = () => eventChannel((emitter) => {
  const tokensApproval = tokens.Approval({ fromBlock: 'latest' });
  tokensApproval.watch((error, results) => {
    const eventType = 'approve';
    const eventTime = new Date().toISOString();
    const eventEthAddress = results.args._owner;
    let eventDescription;
    if (error) {
      console.log(`Approval error ${error}`); // eslint-disable-line no-console
      eventDescription = `ERROR - ${results.args._owner.substring(0,6)} allowed ${results.args._spender.substring(0,6)} to spend ${results.args._value}Ʉ on his behalf`;
    } else {
      eventDescription = `${results.args._owner.substring(0,6)} allowed ${results.args._spender.substring(0,6)} to spend ${results.args._value}Ʉ on his behalf`;
    }

    emitter({ eventEthAddress, eventType, eventTime, eventDescription });
  });

  return () => {
    tokensApproval.stopWatching();
  };
});

function* handleEthEvent(event) {
  switch (event.eventType) {
    case 'approve':
      yield put(ethEventListenerReceiveAction(event.eventEthAddress, event.eventType, event.eventTime, event.eventDescription));
      break;
    default:
      console.log(event); // eslint-disable-line no-console
  }
}

export function* ethEventListenerSaga() {
  yield put(ethEventListenerCreateAction());

  const chan = yield call(ethEvent);
  try {
    while (true) {
      const event = yield take(chan);
      yield call(handleEthEvent, event);
    }
  } finally {
    if (yield cancelled()) {
      chan.close();
      console.log('listening cancelled');
    }
  }
} 
```

And here you go! You can now listen to the stream of events.

## Getting a list of events

In Ethereum, you can get the list of past events, it can kind of act like a kind of storage for the history of your contracts (I won't go into details for this, but some node might not keep the whole history).

How to get the history of events on the contract?

```javascript
import { take, call, put, cancel, fork } from 'redux-saga/effects';
import {
  APPROVAL_HISTORY_GET,
} from './constants';
import { LOCATION_CHANGE } from 'react-router-redux';
import { web3Connect } from 'utils/web3.js';
import {
  receiveApprovalHistory,
} from './actions';
import {
  balanceOfGet,
} from 'containers/Token/actions';
import HumanStandardToken from 'contracts/HumanStandardToken.sol.js';

function* getApprovalHistory(ethAddress) {
  const web3Connection = web3Connect();

  HumanStandardToken.setProvider(web3Connection.currentProvider);
  const tokens = HumanStandardToken.deployed();

  const approvalHistory = [];
  // Getting the list of events from block 0 to latest block, with a filter on _owner as it is an indexed field
  const tokensApproval = tokens.Approval({ _owner: ethAddress }, { fromBlock: '0', toBlock: 'latest' });

  const getApprovalLogPromise = yield call(eventGetPromisified, tokensApproval);

  if (getApprovalLogPromise.err === null || getApprovalLogPromise.err === undefined) {
    for (const results of getApprovalLogPromise) {
      approvalHistory.push({
        trigger: results.args._trigger,
        measurement: results.args._measurement.valueOf(),
        premium: results.args._amount.valueOf(),
        refundedAmount: results.args._refundAmount.valueOf(),
        settled: results.args._timeEnded.valueOf(),
        refunded: results.args._due.valueOf(),
      });
    }
  } else {
    console.log(getApprovalLogPromise.err);
  }

  yield put(receiveApprovalHistory(ethAddress, approvalHistory));
}

// Event promisifier to turn the nasty web3 callback to a promise ES6 form
const eventGetPromisified = (event) => new Promise((resolve, reject) => {
  event.get((error, logs) => {
    if (error) {
      reject(error);
    } else {
      resolve(logs);
    }
  });
});

/**
 * Watches for APPROVAL_HISTORY_GET action and calls handler
 */
export function* getApprovalHistoryWatcher() {
  while (true) { // eslint-disable-line no-constant-condition
    const { ethAddress } = yield take(APPROVAL_HISTORY_GET);
    yield fork(getApprovalHistory, ethAddress);
  }
}

/**
 * Root saga manages watcher lifecycle
 */
export function* approvalHistorySaga() {
  // Fork watcher so we can continue execution
  const getApprovalHistoryW = yield fork(getApprovalHistoryWatcher);

  // Suspend execution until location changes
  yield take(LOCATION_CHANGE);
  yield cancel(getApprovalHistoryW);
}

// All sagas to be loaded
export default [
  approvalHistorySaga,
];
```

Et voila!
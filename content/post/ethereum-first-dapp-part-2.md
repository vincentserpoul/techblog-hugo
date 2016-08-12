+++
date = "2016-08-10T15:00:27+08:00"
title = "Ethereum first dapp - part 2"
description = "Ethereum first dapp - part 2"
tags = [ "ethereum", "dapp", "development" ]
topics = [ "dev", "ethereum", "testrpc" ]
slug = "ethereum-first-dapp-part-2"
+++

# Frontend

## Prepare your folder for your dapp

I will be using https://github.com/mxstbr/react-boilerplate as it's quite nice and I've been playing with React for a bit now.

I will not go into the details of setting this up, it's a totally different topic. 
If you are not familiar with it, it's probably a waste of time for you to read.

## Example web3 component with React

This boilerplate uses immutable, redux and redux-sagas in order to deal with data.
We will use this to connect/dialog with our local node.

### Actions

```javascript
export const ETH_CONNECT = 'app/EthConnect/ETH_CONNECT';

export function ethConnect(web3Provider) {
  return {
    type: ETH_CONNECT,
    web3Provider,
  };
}


```

### reducers

```javascript
import { fromJS } from 'immutable';
import Web3 from 'web3';
import { ETH_CONNECT } from './constants';

const initialState = fromJS({
  web3Connection: false,
});

function ethConnectReducer(state = initialState, action) {
  switch (action.type) {
    case ETH_CONNECT:
      return state
        .set('web3Connection', web3Connect(action.web3Provider));
    default:
      return state;
  }
}

const web3Connect = (web3Provider) => {
  const web3 = new Web3(new Web3.providers.HttpProvider(web3Provider));
  if (web3.isConnected()) {
    return web3;
  }

  return false;
};


export default ethConnectReducer;
```

### selectors

```
import { createSelector } from 'reselect';

/**
 * Direct selector to the ethConnect state domain
 */
const selectEthConnectDomain = () => state => state.get('ethConnect');

/**
 * Other specific selectors
 */


/**
 * Default selector used by EthConnect
 */

const selectEthConnect = () => createSelector(
  selectEthConnectDomain(),
  (substate) => substate.toJS(),
);

const selectEthConnectWeb3Connection = () => createSelector(
  selectEthConnectDomain(),
  (substate) => substate.get('web3Connection')
);

const selectEthConnectIsConnected = () => createSelector(
  selectEthConnectDomain(),
  (substate) => {
    if (substate.get('web3Connection') === false) {
      return false;
    }
    return true;
  }
);

export default selectEthConnect;
export {
  selectEthConnectDomain,
  selectEthConnectWeb3Connection,
  selectEthConnectIsConnected,
};
```

### component itself

```
/*
 *
 * EthConnect
 *
 */

import React from 'react';
import { connect } from 'react-redux';
import { createStructuredSelector } from 'reselect';
import { ethConnect } from './actions';
import { selectEthConnectIsConnected } from './selectors';
import ConnectionStatus from 'components/ConnectionStatus';

export class EthConnect extends React.Component { // eslint-disable-line react/prefer-stateless-function
  // That's when we connect, each time the component is mounted
  componentWillMount() {
    this.props.connect(this.props.web3Provider);
  }

  render() {
    return (
      <ConnectionStatus isConnected={this.props.isConnected} />
    );
  }
}

EthConnect.propTypes = {
  loading: React.PropTypes.bool,
  web3Provider: React.PropTypes.string.isRequired,
  connect: React.PropTypes.func.isRequired,
  isConnected: React.PropTypes.bool,
};

const mapStateToProps = createStructuredSelector({
  isConnected: selectEthConnectIsConnected(),
});

function mapDispatchToProps(dispatch) {
  return {
    connect: (web3Provider) => dispatch(ethConnect(web3Provider)),
    dispatch,
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(EthConnect);
```

And there we go! Our app is up and running and connect to the local node.

In the next step, we will see how to play with this app with more complex interactions, contract functions.
+++
date = "2016-04-11T20:48:27+08:00"
title = "react setup with essential tools"
description = "react setup with essential tools"
tags = [ "react", "tools", "environment", "development", "setup" ]
topics = [ "react", "development" ]
slug = "react-dev-env-setup"
+++

### Working with ES6-7

In order to work with ECMAScript 2015 and even with future implementations of ES, you can use Babel.

Babel is a transpiler, it will convert your ES6-7 to plain ES5 javascript that most browsers (>ie9 most probably) will understand.

To install babel

```shell
npm install -g babel 
```

Then within your javascript project, you can create a .babelrc file with the following content: 

```json
{
  "presets": ["es2015", "stage-0", "react"]
}
```

### React and its surrounding libraries

After starting using React, I realized it was vey good and was surrounded with libraries which makes it even better: redux, immutable, react-router...

I usually try to use these three, for most of the projects I'm working on. Sometimes, it feels way too over engineered, but still, once you understand their use, it's pretty straightforward and logical to implement them.

The most important value added of redux, as well as immutable is the separation of concern. It gives you the power of designing components independantly from eachother.

### React and its tools

In order to help you develop with react and also redux, you have a few tools to help you.

#### eslint for react

```shell
npm install -g eslint-plugin-react
```

#### sublime text plugin

You can install a useful plugins in react, available in the package installer: React ES6 snippets.

#### .eslintrc file for babel and react

You can use this eslintrc file as a starting point.

```json
{
  "extends": "airbnb",
  "ecmaFeatures": {
    "jsx": true,
    "modules": true
  },
  "env": {
    "browser": true,
    "node": true
  },
  "parser": "babel-eslint",
  "rules": {
    "quotes": [2, "single"],
    "strict": [2, "never"],
    "react/jsx-uses-react": 2,
    "react/jsx-uses-vars": 2,
    "react/react-in-jsx-scope": 2,
    "no-console": 0
  },
  "plugins": [
    "react"
  ]
}
```

#### Chrome tools

You can install react devTools in Chrome: 

https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi

You can also install redux devTools in chrome too:

https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd

redux devTools will require you to modifiy the way you call create store:

```javascript
store = (window.devToolsExtension ? window.devToolsExtension()(createStore) : createStore)([...your content here]);
```

You will then be able to play with the different state of your app directly in chrome.
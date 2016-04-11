+++
date = "2016-03-10T20:48:27+08:00"
title = "javascript dev environment setup"
description = "javascript tooling"
tags = [ "javascript", "sublimetext", "tools", "environment", "development", "setup" ]
topics = [ "javascript", "development" ]
slug = "javascript-dev-env-setup"
+++

# NodeJS

Go to the nodejs website and install nodejs latest stable version:

https://nodejs.org/en/download/stable/

# NPM

Go to the npm website and follow the instructions

https://docs.npmjs.com/getting-started/installing-node

# Install nodejs essential packages

npm install -g eslint jshint webpack webpack-dev-server babel-eslint serve 

# Install sublime-text essential plugins

With the help of the package manager, in sublime-text, install the following packages: babel, Sublime-Linter-Contrib-eslint, React ES6 snippets, SublimeLinter-jshint

# Make sure your linter is working

In sublime text, open the console (view > show console) and check if there is any error message.

Alright, let's check if everything is fine. Let's create a test project in a new folder.

Add a .eslintrc file, with this content: 

```
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
    "strict": [2, "never"]
  }
}
```

then create a simple test.js with this content:

```
echo "test"
```

You should see the linter complaining, telling you something is wrong and you need to correct your javascript (check the lower grey line of sublime for the comment).

If you don't see anything, check the console again.

Last resort, if things are still not working, you can go there and troubleshoot: http://www.sublimelinter.com/en/latest/troubleshooting.html
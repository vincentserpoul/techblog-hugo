+++
date = "2016-02-20T20:48:27+08:00"
title = "golang dev environment setup"
description = "go tooling"
tags = [ "golang", "sublimetext", "tools", "environment", "development", "setup" ]
topics = [ "golang", "development" ]
slug = "golang-dev-env-setup"
+++

### Golang environment

install golang

http://www.wolfe.id.au/2015/03/05/using-sublime-text-for-go-development/

Within [sublimetext, from the package manager](/sublimetext-dev-environment), install gosublime, install gooracle.

install go/tools:

```shell
go get -u golang.org/x/tools/cmd/goimports
go get -u golang.org/x/tools/cmd/vet
go get -u golang.org/x/tools/cmd/oracle
go get -u golang.org/x/tools/cmd/godoc
```

install gometalinter (https://github.com/alecthomas/gometalinter)

install interfacer (https://github.com/mvdan/interfacer/)

install gosimple (https://github.com/dominikh/go-simple)

install gocov (https://github.com/axw/gocov)

Here is the package settings I use for gosublime:

```json
{

    // you may set specific environment variables here
    // e.g "env": { "PATH": "$HOME/go/bin:$PATH" }
    // in values, $PATH and ${PATH} are replaced with
    // the corresponding environment(PATH) variable, if it exists.
    "env": {"GOPATH": "/home/go" },

    "fmt_cmd": ["goimports"],
    "ipc_timeout": 5,

    // enable comp-lint, this will effectively disable the live linter
    "comp_lint_enabled": true,

    // list of commands to run
    "comp_lint_commands": [
        // run `golint` on all files in the package
        // "shell":true is required in order to run the command through your shell (to expand `*.go`)
        // also see: the documentation for the `shell` setting in the default settings file ctrl+dot,ctrl+4
        {"cmd": ["golint *.go"], "shell": true},

        // run go vet on the package
        {"cmd": ["go", "vet"]},

        // run `go install` on the package. GOBIN is set,
        // so `main` packages shouldn't result in the installation of a binary
        {"cmd": ["go", "install"]}
    ],

    "on_save": [
        // run comp-lint when you save,
        // naturally, you can also bind this command `gs_comp_lint`
        // to a key binding if you want
        {"cmd": "gs_comp_lint"}
    ]
}
```



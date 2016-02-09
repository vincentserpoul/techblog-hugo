+++
date = "2016-02-09T14:48:27+08:00"
draft = true
title = "golang dev environment setup"
description = "go tooling"
tags = [ "golang", "sublimetext", "tools", "environment", "development", "setup" ]
topics = [ "golang", "development" ]
slug = "golang-dev-env-setup"

+++

# Golang environment

install golang

http://www.wolfe.id.au/2015/03/05/using-sublime-text-for-go-development/

install sublimetext3
    install gosublime
    install gooracle

install go/tools
    go get -u golang.org/x/tools/cmd/goimports
    go get -u golang.org/x/tools/cmd/vet
    go get -u golang.org/x/tools/cmd/oracle
    go get -u golang.org/x/tools/cmd/godoc

install gometalinter (https://github.com/alecthomas/gometalinter)

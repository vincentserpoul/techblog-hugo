+++
date = "2021-03-28T18:39:35+08:00"
title = "Go embed in go1.16"
description = "using go embed in go 1.16"
tags = ["golang", "embed", "static", "simple"]
topics = ["golang", "embed", "static", "simple"]
slug = "go-embed-1.16"
+++

## Golang 1.16 new feature: embed

### What is it about

In previous versions of go, you needed to reach out for an external package in order to embed static content inside your binary.

My go to one was [shurcool/vfsgen](https://github.com/shurcooL/vfsgen), but there were many others worth using.

Go 1.16 brings a new package, "embed", which allows to do that with just the standard library.

### How to do that

The best part of this, it's extremely simple, just see for yourself.
Let's say you want to include an index.html located at the same path of your main.

```golang
package main

import (
	"embed"
	"io/fs"
	"net/http"
)

func main() {
	if err := http.ListenAndServe(":9003", straightforward()); err != nil {
		panic(err)
	}
}

//go:embed index.html
var straightforwardStatic embed.FS

func straightforward() http.Handler {
	return http.FileServer(http.FS(straightforwardStatic))
}
```

### What if you need to embed a subfolder

Now let's say you want to include a full static website, from within a subfolder containing all your files.

With go1.16, a few refactors also happened for the io packages.
Now, a new package io/fs allows you to actually take a subfolder to include.

Let's see how that goes:

```golang
package main

import (
	"embed"
	"io/fs"
	"net/http"
)

func main() {
	if err := http.ListenAndServe(":9003", subfolder()); err != nil {
		panic(err)
	}
}

//go:embed include/*
var static embed.FS

func subfolder() http.Handler {
	subStatic, err := fs.Sub(static, "include")
	if err != nil {
		panic("no folder include present in static")
	}

	return http.FileServer(http.FS(subStatic))
}

```

### Official docs

You can have a look at [official docs](https://pkg.go.dev/embed) to have more details.

Enjoy!

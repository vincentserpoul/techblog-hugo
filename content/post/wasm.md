+++
date = "2018-08-27T23:22:00+08:00"
title = "a taste of wasm"
description = "golang 1.11 is out, let's try wasm"
tags = [ "wasm", "golang" ]
topics = [ "wasm", "golang" ]
slug = "playwithwasm-golang"
+++

Here it is, go 1.11 is out, and with it a new compilation target, Web Assembly!

Let's try it out!

## The test

As dumb as it sounds, I needed a simple usecase to test this out.
I decided to settle for the dumbest possible thing: a for loop incrementing a variable for 100000000 times.

## The git repo

[Here it is](https://github.com/vincentserpoul/playwithwasm)

[I uploaded it as well on surge for you to see without the hassle](https://playwithwasm-golang.surge.sh)

As far as I can see, and if you see something else, I'd be glad to know, **Go is 70x faster than pure javascript**... Crazy!

## Let's all jump to wasm?!

Not yet! A lot of things are missing right now and it is just out.
First issue, the size of the wasm file, which is huge for what it does (thanks to garbage collection included in it, as it is not provided in the wasm VM yet).

## What's next

I will try the same thing with Rust, it should give us the same perf but with a lot smaller size.

*WASM is the future, be ready!*
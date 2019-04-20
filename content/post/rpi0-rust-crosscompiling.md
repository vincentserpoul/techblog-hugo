+++
date = "2019-04-20T19:02:02+08:00"
title = "cross compiling rust from linux to rpi0/alpine"
description = "cross compilation for rust"
tags = ["linux", "alpine", "rpi", "rpi0", "rust", "cross compilation"]
topics = ["linux", "alpine", "rpi", "rpi0", "rust", "cross compilation"]
slug = "alpine-rpi0-rust"
+++

If you read my previous posts, you should now have a raspberry-pi running alpine and docker.

Resources are pretty limited on rpi0, so we are going to deploy programs using rust.
Note: golang could have fit in there as well (and ofc all C and close to machine languages).

## Compiling from NOT a rpi0

Obviously, you don't want to compile from a raspberry pi zero, unless you have all the time in the world and love waiting.
So the solution is to "cross compile" from your local and deploy in the target.

Here, there are two unusual things on the target: it's an arm cpu and the linux distribution is alpine, which is using musl lib c and not glibc.

If you try to compile directly by simply adding the target to your local setup

```bash
rustup target add arm-unknown-linux-musleabihf
cargo build --target=arm-unknown-linux-musleabihf
```

You immediately see errors, you're simply not able to compile here.

## Docker to the rescue

We are not the first ones to do that and some have already figured out an easy way: use docker to do the compilation.

For rpi0, the target is arm-unknown-linux-musleabihf.

The most well known cross compiler image is [this one](https://github.com/emk/rust-musl-builder). Unfortunately, it only takes care of rpi2 and rpi3, which carry armv7.

Fortunately, [another one](https://github.com/messense/rust-musl-cross) is available with the right target available.

## Compiling

This is now very simple:

```bash
docker run --rm -it -v "$(pwd)":/home/rust/src messense/rust-musl-cross:arm-musleabihf cargo build --release
```

Now, either you can use this image to do a multistage build for docker and deploy as a container or simply scp your executable to the rpi0.

Easy!

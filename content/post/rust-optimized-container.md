+++
date = "2021-06-02T18:12:40+08:00"
title = "deploying rust in containers"
description = "leveraging multistage docker build to deploy rust"
tags = ["rust", "container", "docker"]
topics = ["rust", "container", "docker"]
slug = "rust-optimized-container"
+++

### The goal

I am pretty familiar with containers and multi stage deployment have been my go to since quite a while now, for my golang codebases.

Now that I have been picking up rust, deploying my first rust service in production did bring me back to my usual habits of containerization.

We'll be trying to optimize for speed of build and container size.

### The cargo.toml

```yaml
[...]

[lib]
name = "vs"
path = "src/lib.rs"

[[bin]]
name = "vs"
path = "src/bin/vs.rs"

[dependencies]
serde = { version = "^1.0", features = ["derive"] }
serde_json = "^1.0"
serde_with = { version = "^1.9", features = ["chrono"] }
serde_repr = "0.1"
[...]
```

I removed not relevant parts of the Cargo.toml.

You'll pay attention to the lib and bin entries. They allow you to mix a lib and multiple binaries in the same codebase.

I'll be leveraging this in the Dockerfile.

### The Dockerfile

```docker
FROM rust:1.52.1 as builder

WORKDIR /usr/codebase

COPY ./Cargo.toml ./Cargo.toml

RUN sed -i 's#src/bin/vs.rs#fake.rs#' Cargo.toml
RUN sed -i 's#src/lib.rs#fake_lib.rs#' Cargo.toml

RUN echo "fn main() {}" > fake.rs &&\
    touch fake_lib.rs

RUN cargo build --release
RUN rm fake.rs fake_lib.rs

RUN sed -i 's#fake.rs#src/bin/vs.rs#' Cargo.toml
RUN sed -i 's#fake_lib.rs#src/lib.rs#' Cargo.toml

COPY ./ ./

RUN cargo build --bin vs --release

#
FROM gcr.io/distroless/cc-debian10

COPY --from=builder /usr/codebase/target/release/vs /app/

WORKDIR /app

ENTRYPOINT ["./vs"]
```

We first want to cache the compiled dependencies.
To do that, we first copy the Cargo.toml alone and then create a fake.rs and a fake_lib.rs.
Then we tell cargo to look at these new files instead of the previous ones and ask it to compile.
Cargo will compile and cache all the dependencies (as well as your useless fake.rs).
Bingo! All the deps will be compiled and cache in this image layer.

You can now restore the previous values, copy the rest of your codebase and voila, cargo will compile a LOT quicker in your next rebuild as it won't have to compile your deps again (except if they change, of course).

Regarding the container for runtime, instead of using a debian:bullseye-slim, you can go for the distroless image, ending up with a final size of around 20-30MB.
Notice the use of cc-debian10, allowing non statically compiled binaries to run properly.

### The .dockerignore

You simply don't want to copy your huge target repo to the docker image, so you want to add it to the .dockerignore file.
You can also take this opportunity to add any files that should not be exposed, even though it won't appear in the final container, you never know.

```
target
mysecret
Cargo.lock
```

### On from scratch

Usually, I go for a scratch image for my go runtime. The issue here is that some rust dependencies seems to be not so easy to compile statically (at least from what I see now).
If possible, you can go for a musl cross compilation and if that works, go for a scratch container.
Maybe another post!

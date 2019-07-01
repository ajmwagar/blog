---
title: "Oxidizing my Workflow: Writing a Wayland window manager in Rust - Part 1: Setting Up"
author: Avery Wagar
date: 2019-06-30
draft: false
toc: false
tags:
  - Rust
  - Wayland
  - Programming
  - Multi-threading
  - Concurrency
---

In this series, we will be creating a usable window manager for Wayland in Rust.

Here are some goals for our window manager:

- Fast
- Rounded corners on windows
- Tiling + Floating window support
- `TOML` File configuration
- Parallel / Concurrent programming model

**Note: I have installed Rust using the `rustup` installer. Please install [rustup](https://rustup.rs) before continuing.**

# Step one: Set up the project w/ Cargo

```bash
cargo init --lib oxy # oxy can be replaced with whatever you want to call your window manager
```

This creates a new folder called `oxy` and adds a `Cargo.toml` file and a `lib.rs` file in `./oxy/src/`.

Now, let us create the binary files which will be available to users at runtime.

```bash
cd oxy
mkdir ./src/bin
touch ./src/bin/oxy.rs # This is the actual window manager binary
touch ./src/bin/oxymsg.rs # This binary lets us send messages/commands to oxy at runtime
```

A little breakdown of each file:
- `oxy.rs`: The actual window manager, this gets called at bootup and starts the window manager
- `oxymsg.rs`: This binary lets us send `oxy` messages and commands at runtime, this is helpful for re-configuring on the fly.

In both `oxy.rs` and `oxymsg.rs` add this:

```rust
fn main() {
    println!("Hello, Wayland!");
}
```

This allows for `Cargo` to actually build the binaries. We need to be able to build so we can do testing. We will change these file dramatically later on in the series. 

# Step two: Set up a place for tests/benchmarks

Since performance is a goal for our window manager, we should monitor our performance over time with benchmarks and prevent regressions with unit tests. Luckily for us, Rust has 1st party support for both tests and benchmarks.

It just takes a second to setup:

```bash
mkdir ./tests ./benches
touch ./tests/common.rs ./benches/common.rs
```

In `./tests/common.rs` add this:

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}
```

Now we can do `cargo test`. It should output the following:

```bash
$ cargo test
   Compiling oxy v0.1.0 (/home/ajmwagar/usr/dev/git/rust/oxy)
    Finished dev [unoptimized + debuginfo] target(s) in 1.57s
     Running target/debug/deps/oxy-eb66d2a31f6edeb1

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/oxy-8a8962f5d923f1bf

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/oxymsg-b94e3c7436b46996

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-7af7b5cd849c06d8

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests oxy

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

```

Great! Now we having unit testing set up. We will get into benchmarking later in the series (once we have code to benchmark).

---

Now that we have our project set up, we can get to the actual code...

**Stay tuned for part two!**

This commit: [98f36f1](https://github.com/ajmwagar/oxy/commit/98f36f100bd2ed5dc61bc51782cdc72a99638d20) is a reference to where you should be after this post.


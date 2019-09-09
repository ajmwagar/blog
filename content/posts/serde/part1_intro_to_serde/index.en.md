---
title: Intro to Data Serialization with Rust & Serde
author: Avery Wagar
date: 2019-08-03
draft: false
toc: false
tags:
  - Rust
  - Programming
  - Data Serialization
  - Serde
---

Transmitting data across processes is hard, epically when the structure needs to remain intact.

For example, JSON (JavaScript Object Notation) is a 1st class citizen on the web. Converting a JavaScript Object in to JSON is trivial (`JSON.stringify(obj)`). However in lower level languages such as Rust and C++, converting Data Structures to a transmittable format is not nearly as easy. This is where data serialization comes in.

> **Serialization (noun)**: In computer science, in the context of data storage, serialization is the process of translating data structures or object state into a format that can be stored or transmitted and reconstructed later. 


## Enter Serde

In Rust, we can use a library called `serde` (**ser**ialization/**de**serialization) to easily do data serialization and deserialization.

To install `serde` simply add the following to your `Cargo.toml`:

```toml
[dependencies]
serde = "1"
```

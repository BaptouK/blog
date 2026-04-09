+++
title = "Discovering Rust"
date = 2026-03-15
description = "My first impressions learning Rust."
[taxonomies]
tags = ["rust", "programming", "learning"]
+++

Rust is a fascinating language. The borrow checker is confusing at first, but once you understand the ownership concept, everything clicks.

## What I liked

- Memory management without a garbage collector
- The powerful type system
- Compiler error messages, incredibly clear

## First program

```rust
fn main() {
    let message = String::from("Hello world!");
    println!("{}", message);
}
```

The learning curve is steep but worth it. Next up: lifetimes.

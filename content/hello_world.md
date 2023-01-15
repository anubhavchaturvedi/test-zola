+++
title = "Understanding Atomics"
description = "Explains the basic of atomics in Rust. Comes with a bonus: spinlock implementation."
date=2018-05-30

[taxonomies]
categories = ["Data Type"]
tags = ["atomic", "utils", "bla"]

[extra]
author = "Bruno Corrêa Zimmermann"
relative_posts=[
    {label="Other about atomics", url="/relatively_simple/other-hello-world"}
]
+++

# Abstract

In this post I will explain what are atomics and how to use them in rust.

# "What is an 'atomic' anyway?"

Atomics are data types on which operations are indivisible,
or the indivisible operations by themselves. Indivisible here means that
the side effects of the operations are only observable when the operation
is done. Generally, machines provide read and write operations over its
native integer both indivisible and divisible.

## `AtomicBool`

It is very simple to create one:

```rust
let atomic_bool = AtomicBool::new(true);
```

hjsdjhfsd

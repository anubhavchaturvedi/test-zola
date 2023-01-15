+++
title = "Understanding Atomics"
description = "Explains the basic of atomics in Rust. Comes with a bonus: spinlock implementation."
date=2017-05-30

[taxonomies]
categories = ["Data Type"]
tags = ["atomic", "utils", "bla"]

[extra]
author = "Anubhav Chaturvedi"
+++

# Abstract

In this post I will explain what are atomics and how to use them in rust.

# "What is an 'atomic' anyway?"

Atomics are data types on which operations are indivisible,
or the indivisible operations by themselves. Indivisible here means that
the side effects of the operations are only observable when the operation
is done. Generally, machines provide read and write operations over its
native integer both indivisible and divisible.

# Atomics in Rust

Currently (rust nightly 1.28 and stable 1.26) these atomic data types were
stabilized: `AtomicPtr<T>`, `AtomicUsize`, `AtomicIsize` and `AtomicBool`.
They are all located in `core::sync::atomic` (or `std::sync::atomic`).
To explain, I will pick the simplest one: `AtomicBool`.

## `AtomicBool`

It is very simple to create one:

```rust
let atomic_bool = AtomicBool::new(true);
```

Now let's make an atomic read. But wait... uh oh... `AtomicBool::load`
accepts two arguments: san immutable reference to `self` and another
argument of type `Ordering`. The reference to `self` is easily understandable
but what about `Ordering`? `Ordering` is a type which determinates the...
the... order? Yes, the order related to other operations and other threads.
Let's see the definition of `Ordering`:

```rust
pub enum Ordering {
    Relaxed,
    Release,
    Acquire,
    AcqRel,
    SeqCst,
    // some variants omitted
}
```

There are, roughly speaking, two kinds of CPUs related to orderings: the weak
and the strong. The weak processors have naturally weak guarantees on the
orderings, while the strong processors have naturally strong guarantees on the
orderings. It is generally low-cost for strong-processors to perform `Acquire`,
`Relase` and `AcqRel`, while they are high-cost for weak-processors. For all
of them `Relaxed` is low-cost and `SeqCst` is high-cost. Examples of weak-
processors are ARM-archs, and examples of strong-processors are the ones
x86/x86_64-archs.

The only valid variants for `load` to be called with are: `Relaxed`, `Acquire`
and `SeqCst`. Don't worry, the other variants will be explained later. `Relaxed`
is somewhat like "the order does not matter at all". This allows both
the compiler and the CPU to reorder the operation. `SeqCst` means "sequentially
consistent"; it should not be reordered at all! Everything before it happens
before it, and everything after it happens after it.

`Acquire` is a bit more complex. It is the "complement" of `Release`.
Everything (generally `store`s) that happens after the `Acquire` stays after
it. But the compiler and the CPU are free to reorder anything that happens
before it. It is designed to be used with `load`-like operations when acquiring
locks.

Let's see an example with `AtomicBool::load`:

```rust
use std::sync::atomic::{
    AtomicBool,
    Ordering::*,
};

fn print_if_it_is(atomic: &AtomicBool) {
    if atomic.load(Acquire) {
        println!("It is!");
    }
}
```

Let's jump into the next operation: `store`. `store` accepts a reference to
`self`, the new value (of type `bool`), and an `Ordering`. The valid
`Ordering`s are `Relaxed`, `Release` and `SeqCst`. `Release`, as said before,
is used as a pair with `Acquire`. Everything before `Release` it happens before
it, but the compiler and the CPU are free to reorder anything that happens
after it. It is intended to be used when releasing a lock.

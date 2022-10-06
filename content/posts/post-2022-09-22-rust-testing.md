+++
title = "Rust for Rustaceans #6: Testing"
date = 2022-09-22

+++

## Introduction

In this chapter we will go beyond `#[test]` and what we can find in `/test`
directory. First part covers rust testing mechanisms, and the second goes
into things like benchmarking, linking and fuzzing.

## Rust Testing Mechanisms

When we run `cargo test` what cargo does is it just passes `--test` flag to
`rustc`. This tells compilers to produce a binary that runs all the unit
tests. This flag enables `cfg(test)` and more importantly it generates a
**test harness** with a `main` function that runs all the tests.

### The Test Harness

Every `#[test]` annotation causes to transform every function annotated by it
into a **descriptor**. It then exposes the path of each descriptor to the
generated `main` function. This *descriptor* includes information like
a test name, ano other importation fact like inclusion of `#[should_panic]`.
Then the test harness iterates over all these tests and runs them, captures
their results and prints them. This means that the binary also include CLI
arguments parsing logic, running tests in parallel, collecting the results,
prettyr printing them and so on. Integration tests that follow the same path
except they have a separate compilation per each file in `/tests`.

You can use different test harnesses than default one by opting out using
the following entry inside `Cargo.toml`.

```toml
[[test]]
name = "custom"
path = "tests/custom.rs"
harness = false
```

In this situation we are expected to write our own `main` function that runs
our test code.

> `cargo test` just runs the binary that is compiled using `rustc --test`
> underhood. This binary can take different options. These options can be
> passed after `--` in `cargo test --`.

### `[cfg(test)]`

When we run `cargo test` we set `test` configuration flag that is available
for us inside code to use for conditional compilation. We can decide
to have some code be compiled only for tests.

> Isn't this obvious?

Usually we mock things by defining generic code through generics. But sometimes
we don't want to provide `trait`s to our users and we want to use a regular
types instead. In that situtaiton we could have two types defined with
identical interfaces but on specifically for test purposes and other one
for everything else using `[cfg(test)]`.

### Test-Only APIs

> This is cool way of using `[cfg(test)]`.

Imagine we implement a `HashMap` that underhood uses some other types that
we have defined in different modules. We would like to test that the
internal state of our HashMap is correct but in order to do that we have to
access internal type. But the fields of that internal type are private.
What we can do is for that internal type define public functions that
allow us to access that type but annotate them with `[cfg(test)]`.

```rust
struct RawTable {
    // private
    buckets: Vec<?>
    
}

impl RawTable {
    #[cfg(test)]
    pub(crate) fn buckets(&self) -> &[Bucket] {
        &self.buckets
    }
}
```

Why not just use `pub(crate)` and call it a day? Because by doing that we expose
the private things and even though only our crate has access to them we could
still make a mistake and by messing around with that private thing cause
invariants to break.

### Bookkeeping for Test Assertions

We can use conditional compilation to add fields to our types that are just
for the sake of tracking changes of the instance and check it during our tests.
The example from the book shows implementation of `BufWriter` that
checks number of calls to `write` by adding a counter field to the type but
only under tests.

```rust
struct BufWriter<T> {
    #[cfg(test)]
    write_through: usize,
}
```

### Doctests

Each doctest is compiled is compiled in its own dedicated crate that
has our main crate as a dependency thus it has only access to the public API.
We can even hide parts of code inside a doctest.

```rust
/// Adds two numbers.
/// ```
/// #let x = 1;
/// #let y = 2;
/// assert_eq!(add(x, y), 3);
/// ```

fn add(x:usize, y:usize) -> usize {
    x + y
}
```

```rust
/// Adds two numbers.
/// ```compile_fail
/// # struct MyNonSendType(std::rc::Rc<()>);
/// fn is_send<T: Send>() {}
/// is_send::<MyNonSendType>();
/// ```
```

## Additional Testing Tools

### Linting

Use `clippy` and enable all `rustc` lints.

```rust
// lib.rs or main.rs
#![deny(
    warnings,
    unused,
    missing_debug_implementations,
    rust_2018_idioms,
    rust_2021_compatibility,
    nonstandard_style,
    future_incompatible,
    clippy::all
)]
#![forbid(unsafe_code)]
```

### Test Generation

#### Fuzzing

Fuzzing means generate random inputs to your program and see if it crashes.

```rust
libfuzzer_sys::fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = url::Url::parse(s);
    }
});
```

`fuzz-test`, `cargo-fuzz` and `arbitrary`.

#### Property-Based Testing

`proptest`

### Test Augmentation

Sometimes tests might be working for a very long time and suddenly they crash
despite the fact the code stayed unchaged. This ussually occurs due to race
conditions or undefined behaviors. These are hard to detect but we can
run our test code using `miri` (`cargo miri test`) which is interpreter
for rust intermediate representation. When we do that we get much more
information about what has happened. For concurrent problems use `Loom`.

### Performance Testing

Performance testing has many problems.

#### Performance Variance

How our functions perform can different even when we run them on the same
hardware. High temperature can cause a CPU to throttle, software updates,
driver updates, kernel changes, and many other things can affect our
performance measurments. To fight that we ususually want to run our benchmarks
many times and analyze their distribution.
Use `criterion`, it reduces the noise by doing just that. Of course it cannot
remove problem of using a different hardware.

#### Compiler Optimizations

To avoid compiler optimizing your code away try to use `std::hint::black_box`
function.

```rust
let mut vs = Vec::with_capacity(4);
let start = std::time::Instant::now();
for i in 0..4 {
    black_box(vs.as_ptr());
    vs.push(i);
    black_box(vs.as_ptr());
}
println!("took {:?}", start.elapsed());
```

### I/O Overhead Measurement

Avoid IO code in your benchmark code!

## Summary

And that is it. `[cfg(test)]` can be used in many cool ways to help us
write better test code.

[Previous chapter - Project Structure](/posts/post-2022-09-11-project-structure-rust)

[Next chapter - Macros](/posts/post-2022-10-06-rust-macros)

+++
title = "Error Handling in Rust (Rust for Rustaceans #4)"
date = 2022-08-25

+++

## Error Handling

This chapter as the title suggests focuses on errors. We will go through
different topics like how to represent errors with some less ordinary
approaches while later on, we will focus on handling the aforementioned errors.
What is important is that this chapter focuses on fundamental mechanisms
without diving into more opinionated ways of error handling represented
by different 3rd-party crates. This is because the rust community has yet
to settle on how to deal with errors in a uniform way.

> I am not sure I completely agree with that. It seems that most crates either
> use barebones `std` way of doing things, i.e., implementing `Error` on their
> own and propagating errors using `dyn Error` values or they use 3rd party
> crates like
> `anyhow` and `thiserror` in particular. But there are other attempts like
> `eyre`.

## Representing Errors

How we represent errors should be decided by looking at what the clients
will most likely do when such an error is encountered.

There are two main ways of representing errors now either through a
**sum-type** aka `enum` or through **erasure**. **Erasure** in this context
I believe means **type-erasure** which means that the exact type is
hidden behind a trait as a *trait object*.

This means we can either enumerate possible error conditions for the caller
and let them decide what to do since they can distinguish between them or
we can just go with a single *opaque* error hidden behind a trait.
**Opaque** here means that we cannot *"see"* the exact details of what
happened as they are obscured.

> A lot of fancy words here for saying that when you have multiple error
> conditions that user might want to distinguish between you want to use `enum`,
> and if you have no such concerns or there is only one possible error
> use a type like `struct DivisionByZeroError` and probably hide it behind a
> trait.

### Enumeration

Enumeration ought to be used when the user has/wants to distinguish between
different error conditions. Jon gives as
an example a method similar to `std::io::copy` for which we provide
two streams as arguments. One to read from, one to write to.

```rust
pub fn copy<R: ?Sized, W: ?Sized>(reader: &mut R, writer: &mut W)
    -> ???
    where
    R: Read,
    W: Write
{
}
```

We would like to be able to distinguish between which stream has failed and we
certainly can expect that the clients of this method would like to do that as
well. We could create an enumeration like the below.

```rust
pub enum CopyError {
    ReadingError(std::io::Error),
    WritingError(std::io::Error),
}
```

> Actually I find it funny that the `std::io::copy` does not provide a way
> to distinguish between which stream caused the problem. It only
> returns `std::io::Error`, for which neither variant gives information
> about which stream has failed.

### Making your own error type

In order for the user-defined error type to play nicely with the *Rust*
ecosystem we need to do three things first (maybe four).

The first thing is to implement `std::error::Error` trait that provides
methods for error introspection. The main method of interest is `Error::source`
which provides a way to find the underlying cause of the error. This is used
to print a backtrace that displays a trace all the way back to the error's root
cause.

The second is to implement `Debug` and `Display` traits so that the callers
can meaningfully print the error. `Debug` and `Display` is required by `Error`
traits, so actually we need to implement first `Debug` and `Display`.

```rust
pub trait Error: Debug + Display {
    // ...
}
```

Jon suggests that `Display` implementation should provide a one-line
description of what went badly that can be easily ingested into other
error messages. Also, the `Display` format **should be all lower-case** and
**without trailing punctuation**.

`Debug` should be more descriptive containing all the information that might
help in tracking the root cause of the problem.

*Third* thing is to implement both `Sync` and `Send` for your error types so
they can be shared across thread boundaries.

At last, Jon says that error types should be `'static`, i.e., they should not
contain non `'static` references which might cause issues when propagating
errors since we will be popping the error down the stack. Also in order to use
`Error::downcast_ref` the underlying type must be `'static`.

```rust
#[derive(Debug)]
enum CopyError {
    ReaderError(std::io::Error),
    WriterError(std::io::Error),
}

impl Display for CopyError {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), std::fmt::Error> {
        match self {
            CopyError::ReaderError(ioerror) => {
                f.write_str(&format!("reader failed {}", ioerror))
            }
            CopyError::WriterError(ioerror) => {
                f.write_str(&format!("writer failed {}", ioerror))
            }
        }
    }
}

impl std::error::Error for CopyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            CopyError::ReaderError(e) => Some(e),
            CopyError::WriterError(e) => Some(e),
        }
    }
}
```

## Opaque Errors

Opaque errors can be used when the information on what is the cause of the problem
cannot meaningfully help the client recover from the problem. In that case,
you can even return something like a `Box<dyn Error + Send + Synd + 'static>`
just so the user can bubble the error further to log it for example.

What are the benefits of type-erased errors?

- Errors from different sources can be easily combined (we don't need to
  convert anything because we return errors behind a trait), `?` operator
  will work on everything in that case without having to resort to conversion.
- It might happen that even though the user does not care about most possible
  errors that might be returned by a function, they might care about a single
  one out of many. In such situation, they can easily use `Error::downcast_ref`
  to check the underlying type of the error.
  **IMPORTANT**: If our error type is not `'static` we remove that ability
  to *downcast* from the user.

Jon then discusses whether typed-erased errors are part of public and stable
API. There is no consensus but Jon says that if your API returns `Box<dyn Error
...>`, then it does so for a reason and you should not rely on `downcast_ref`.

In essence it means that you can use `Error::downcast_ref` but you
probably shouldn't.

## Special Error Cases

Jon states that one should not use `Result<T, ()>` interchangeably with
`Option<T>` as they have different meanings. While `Result<T, ()>` means
that the function failed in the case of `Err(())`, `Option<T>` means that
functions simply have nothing to return.

- `Err(())` must probably be logged, handled exceptionally etc.,
- `None` might not be something that must be necesarilly handled.

> Hmm? If I call a `head` method on a `list` and it returns `None` I have to
> handle it even though it is not exceptional I guess. I think there is a
> wording dynamic at play. When `None` is returned from the `head` method
> even though we most usually are going to handle it, it isn't something
> unexpected. Rather just one of possible outcomes. When `Err` is returned
> it means that something really went wrong and we must do something. That
> is why `Result<T, E>` has `[must_use]`, and `Option<T>` does not.
> A better example would be `pop`ping out of a `Vec`tor. You might not care
> if there is something inside the vector to pop so you can disregard the
> `None`.

For special functions that never return since they run in infinite loop, for
example, there is a special **never** type `!`. It represents a value
that cannot ever be generated.

```rust
fn infinite_loop() -> Result<(), !> {
    loop {
    }
    Ok(())
}
```

Finally, there is a special `std::thread:::Result` type which does not
*type-erase* to `dyn Error` but to `dyn Any`. The reason is that the `Err`
variant of that type is produced only in response to a `panic`, as `panic`
can return `Any`. This happens if you try to join a thread that has panicked.

## Propagating errors

`?` does call `Into::into` on the unwrapped value.

Simplified version looks like this.

```rust
match propagated_result {
    Ok(v) => v,
    Err(e) => return Err(e.into()),
}
```

This is how `try!` macro expands.

```rust
fn fail() -> Result<(), ()> {
    try!(Err(()))
}

// macro expanded
fn fail() -> Result<(), ()> {
    match Err(()) {
        ::core::result::Result::Ok(val) => val,
        ::core::result::Result::Err(err) => {
            return ::core::result::Result::Err(::core::convert::From::from(err));
        }
    }
}
```

This is why returning `Box<dyn Error>` is so appeling because we don't need
to provide implementations of `From`.

> Actually `?` now uses `Try` trait which gives more possibilites because it
> might apply to types other than `Option` and `Result` that have similar
> monadic semantics.

### From and Into Note

While `From` and `Into` exist together initially for historical reasons and
now are kept together for backward compatibility there is advantage to having
them both coexisting. Both of them together provide
good ergonomics and one can be better utilized over another depending on
circumstances.

```rust
fn(impl Into<Foo>)
fn<T>(T) where Foo: From<T>
```

## Summary

We can use `enum` types for errors for which different conditions that
set them off might be useful to the user. For cases other that that
using **type-erasure** and **opaque types** provide better ergonomics
and can be definitely useful.

Creating error types that work well with rust ecosystem
involves implementing `Error` (with `source` if possible) + `Display` +
`Debug` + `Sync` + `Send` + `'static` if possible.

Don't confuse `Result<T, ()>` with `Option<T>`. They convey different meaning.

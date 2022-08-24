+++
title = "Designing Interfaces in Rust (Rust for Rustaceans)"
date = 2022-08-24

+++

## Designing interfaces in Rust

This is my summary of the 3rd chapter of "Rust for Rustaceans" by Jon Gjengest.
In this chapter Jon explains how to make API in Rust as pleasant and idiomatic
as possible. Under *pleasant* and *idiomatic* lie a couple of characterstics
namely:

- *unsurprising*,
- *flexible*,
- *obvious*,
- and *constrained*.

Together with Jon we will dive into each of them in more details.

## Unsurprising

Unsurprising is what Jon calls an API that adheres to the Principle of Least
Surprise. In essence it means that the API should be intuitive and its user
should be able to use it without dwelving into implementations details.

### Naming Practices

When it comes to things that ought to behave the same, their names should
indicate that and vice versa.

How does the *unsurprising* API look like? Rust example (and the one from)
the books is that if `iter` method on Vec returns `Iterator` where `Item=&T`
then it would be intuitive that same named methods but in different collection
types should behave exactly the same and this mean runtime and type behavior.

> Even though type signatures can be the same (and that is good because they
> will be assumeed to be by the user), their runtime behavior could be
> different so it is important to make sure it is also the same.

```rust
impl Vec<T> {
    pub fn iter(&self) -> Iter<'_, T>;
}

impl<T> [T]
    pub fn iter(&self) -> Iter<'_, T>;
}
```

Where both `Iter<'_, T>` types that are returned from `iter`
implement `Iterator<Item=&T>`.

So in essence naming is important and should be consistent when similar
behavior (both runtime and type-level) is expected.

### Common Traits For Types

In another section Jon suggests to implement common traits that users will
expect anyway.
This is important because the user of our API cannot implement standard
traits for our types due to the *orphan rule* (mentioned in the previous
chapter, both the trait and the type are foreign to the client code).
They would need to wrap our type (newtype-pattern), which while
feasible is annoying at least.

What are these traits? They are:

- `Debug`,
- `Sync` + `Send` (users expect types to work in `Mutex`'s and `Arc`s),
- `Default`,
- `PartialEq`, `Eq`, `PartialOrd`, `Ord`, `Hash`,
- and finally `serde::Serialize` and `serde::Deserialize`.

### Ergonomic Trait Implementations

Implement traits also for reference types of your type! In essence if you have
the following.

```rust
impl<T> SomeTrait for T;
```

Provide also the following.

```rust
impl<T> SomeTrait &T;
impl<T> SomeTrait for &mut T;
impl<T> SomeTrait for Box<T>;
```

### Wrapper types

Wrapper types are the candidates for implementing `AsRef<InnerType>`,
`Deref<InnerType>`, and `Into<InnerType>`. This makes working with such types
much smoother as `Deref` makes it easier to use methods from the inner type,
`AsRef` lets you use `&WrapperType` value where `&InnerType` is expected,
and finaly `Into` lets you simply convert between the two.

Similar thing could be said for `Borrow`. `Borrow` is very similar to
the `AsRef` trait but it provides additional requirement that additional traits
must be behaving the same for both the borrowed type and the owner type.

> In particular `Eq`, `Ord` and `Hash` must be equivalent for borrowed and owned
> values: `x.borrow() == y.borrow()` should give the same result as x == y.

## Flexible

This section talks about how to maintain a good contracts throughout your API
without involving unnecessary restrictions. It is important because
lessening restrictions and expanding promises that not usually require to break
the API.

For example:

```rust
fn frobnicate1(s: String) -> String;
fn frobnicate2(s: &str) -> Cow<'_, String>;
fn frobnicate3(s: impl AsRef<str>) -> impl AsRef<Str>;
```

Function one requires us to break the API if we would like to change our
promise by for example returning a slice instead of allocated String. The
same goes for the argument.

Functions two lifts some of the restrictions but still if we would like to
change the types we would need to break API.

The third function finally uses actual traits which means we could switch
underlying returned types and also the client can use different argument types.

While the third one seems to be the best it all still depends on the
requirements. If inside the function you need in the end to have an allocated
buffer, the it is probably better to use `String` for the argument type
anyway to avoid unnecessary allocation.

### Generic Arguments

So should we use only **generic** parameters in our APIs. Reading API
that involves a lot of generics can be at best a little obscuring, then
intidating and at worst impossible to understand for non-experts (like me).

Jon suggests that you should use generic in you API if you can think of
many different types that could be used as an argument. If you cannot think
of any, don't bother I guess.

### Object Safety

In general it should be worthwhile effort to sacrafice some egonomic if
this will allow your trait to be **Object Safe**. I will write more on that
in some other post.

### Borrowed vs. Owned

This sections is rather obvious. When you need owned data, use owning types
for parameters, i.e., make callers provide owned data. Otherwise use
references.

### Fallible and Blocking Desctructors

Okay this is some crazy stuff that I don't even want to mess with right now.
I will expand this later probably.

## Obvious

Make your interfaces as easy as possible to understand and as hard as possible
to misues. You can help with first by providing good documentation and with
second by taking advantage of the type-system.

### Documentation

1. Clearly document any cases where your code may do something unexpected.
    - relying on some outside state,
    - possibility of panicking,
    - etc.
2. Include e2e usage examples. It is even more important that regular function
   and type examples because it shows how everything fits together.
3. Organize the documentation by taking advantage of good modularization
   through modules. Use intra-documentation links, external links, etc.
4. Explain concepts, data structures, and algorithms.

### Types System Guidance

1. Use Semantic-Typing (basically simple types, i.e., types that wrap primitives
and probably constrain their domain).
2. Use `enum`s instead of `bool`s, so arguments order cannot be mistaken and
arguments don't neet to be assigned to names to know what is going on.
3. Use state machines (you can use `std::marker::PhantomData` to simulate).
4. Use `#[must_use]`.

## Constrained

Be ware of some less obvious changes to your API that might be breaking
backward comaptibility.

### Type Modification

Use `pub(crate)` and `pub(in path)` modifiers whenever possible!

ADDING BOTH PRIVATE AND PUBLIC FIELDS MIGHT BREAK COMPATIBILITY.

```rust
pub struct UNIT { pub field: bool };

fn is_true(u: lib::Unit) -> bool {
    matches!(u, Unit { field: true });
}

// if we add private field now the pattern match will fail
pub struct UNIT { pub field: bool, x: bool };

// BROKEN NOW
fn is_true(u: lib::Unit) -> bool {
    matches!(u, Unit { field: true });
}
```

You can disallow use of implicit constructors by using `#[non_exhaustive]`
which mitigates these problems.

### Trait Implementations

In general be careful with adding trait implementations for existing types
if the trait or type is foreign. You sohuld either add them immediatly when
the library is in its infancy. If you are doing it later you may break code
of some people.

Most changes to the existing traits are also breaking changes. be careful.

### Hidden Contracts

Re-exporting foreing types ties you library with foreign library. If someone
uses bot your API and that foreign API if there is a mismatch between the one
use by you and directly by them you will have problems. The best way to avoid
that is by wrapping such types and only then exporting them.

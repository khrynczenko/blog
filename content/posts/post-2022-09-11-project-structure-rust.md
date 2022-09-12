+++
title = "Rust for Rustaceans #5: Project structure "
date = 2022-09-11

+++

## Introduction

This chapter as the title suggests focuses on structuring our projects. This
chapter goes beyond `cargo new` and looks into things like *features*,
reducing compile times, etc.

## Features

To customize our project we can use *features*. Features are just like build
flags that create can pass to their dependencies to turn on some optional
functionality. We use the a lot for example when we want our dependency to
come with `serde` `Serialize` and `Deserialize` implementations for the
types it comes with.

When we create our own project that might become a dependency of some other
projects we use features in three ways:

- to enable optional dependencies,
- to conditionally include optional components of a crate,
- and to augment the behavior of the code.

**All of these are additive**. This is important because in general adding
a new dependency or new feature **should not break the compilation**. When
a crate in its dependency tree has two occurrences of the same dependency
with two different sets of features, it will compile this dependency with
the union of these features. If features would be mutually exclusive all hell
would break loose.

### Defining and including features

Below we define `derive` features that if present enables `syn` dependency.

```toml
# Upstream package
[package]
name = "foo"

[features]
derive = ["syn"]

[dependencies]
syn = { version = "1", option = true }
```

```toml
# Downstream package
[package]
name = "bar"

[dependencies]
foo = { version = "1", features = ["derive"]}
# dependency `foo` will compile with `syn`
```

Cargo allows to define a default set of features that a client can opt out
from.

```toml
# Upstream package
[package]
name = "foo"

[features]
derive = ["syn"]
default = ["derive"]

[dependencies.syn]
version = "1"
default-features = false
features = ["derive", "parsing", "printing"]
optional = true
```

One thing to note is that what is on the right side of a feature is
another feature. But if the name of that feature is a dependency marked
as **optional** (`optional = true`), then that feature corresponds
directly to adding/removing that dependency.

### Using features in your crate

You have to make sure that if the feature is turned off, the code
that uses that feature (dependency) is also removed from the compilation.
To achieve that we use **conditional compilation**. We do that through
annotations that activate/deactivate a particular piece of code.
The too for the job is the `[cfg]` attribute, and its nephew is the `cfg!` macro.

The most obvious example of using `[cfg]` attribute is in front of
test modules.

```rust
# ...

#[cfg(test])
mod tests {
    # ...
}
```

We compile test code only when we compile in `test` mode. It would be
a complete waste of resources to add them into release binary and moreover
a potential security hole.

## Workspaces

While in C++ each source file is compiled in a separate translation unit,
when it comes to Rust, each crate is compiled as a single compilation unit.
Jon explains that `rustc` more or less treats a crate as one big source file.
This is particularly painful because it means that a single change anywhere
inside a crate causes the reevaluation of the whole crate by the compiler. It
cannot evaluate the changes in isolation.

Workspaces can help with that problem by introducing a concept of multiple
subcrates connected by a top-level `Cargo.toml` file. A workspace
is a collection of crates.

```toml
# top-level Cargo.toml
[workspace]
members = [
    "foo",
    "bar/one",
    "bar/two",
]
```

Then within the crate inside the workspace, we can specify dependencies
using relative paths.

```toml
[dependencies]
one = { path = "../one" }
two = { path = "../../two" }
```

This might cause even better compile times if you compile from scratch
because optimizations will be local to the crates, which on the other
hand might cause performance to suffer.

*Be careful when specifying intra-workspace dependencies if they are
to be publicly available.*

## Project configuration

Check [cargo manifest](https://doc.rust-lang.org/cargo/reference/manifest.html)
to know what to put inside your `Cargo.toml` metadata.

### Build configuration

`[patch]` allows you to quickly replace a depndency with a one that is present
in a different place, for example you have a local version with some changes
that you want to test out perhaps for debugging purposes.

```toml
[patch.crates-io]
regex = { path = "/home/me/regex" }
serde = { git = "https://github.com/serde-rs/serde.git", branch = "faster" }
# patch a git dependency
[patch.'https://github.com/jonhoo/project.git']
project = { path = "home/jon/project" }
```

If you have multiple versions of the same dependency in your dependency
tree you can patch each on of them.

```toml
[patch.crates-io]
nom4 = { path = "...", package = "nom"}
nom5 = { path = "...", package = "nom"}
```

Cargo will look inside a tree and notice two different major versions and
replace them accordingly.

### Package vs crate

What is a difference between package and crate?
Crate refers to a source code with a root module like `lib.rs` or `main.rs`
while package consists of that plus metadata and cab include a collection
of crates.

### [profile]

`[profile]` allows to pass options to the Rust compiler to affect
how our code is compiled.
These option fall into three categories: *debugging options*, *performance
options*, and *user options* (options that change behavior of the code.

Three primary performance options are:

- `opt-level` - optimization level (O0, O2, O3, Os),
- `codegen-units` - tells into how many concurrent tasks a compilation
of a crate should be splitted (code optimization might suffer),
- `lto` - link time optimization, particularly usefill when compilation
includes many crates. By default `lto` is done only for all codegen units
within a single crate but not across different crates.

For the debugging options we have:

- `debug` - tells whether to include debug symbols in the binary,
- `debug-assertions` - enables `debug_assert!` macro,
- `overflow-checks` - enables runtime overflow checks.

You can override such options for dependencies as well which might come handy
if for example debugging is quite slow due to particular dependency.

```toml
[profile.dev.package.serde]
opt-level = 3
```

## Conditional compilation

### Operating system options

```rust
[cfg(target_os = "windows")]
[cfg(target_os = "linux")]
[cfg(target_os = "macos")]
[cfg(any(target_os = "linux", target_os = "windows"))]
[cfg(target_family = "windows")]
[cfg(target_family = "unix")]
```

### Tool options

Some tools set custtom options that let you customize compilation.

```rust
#[cfg_attr(miri, ignore)]
#[deny(clippy::all)]
```

### Architecture options

Options for different ISAs (or simply speaking CPU architectures),

```rust
#[cfg(target_arch = "x86")]
#[cfg(target_arch = "mips")]
#[cfg(target_arch = "aarch64")]
#[cfg(target_feature = "sse2")]
#[cfg(target_endian = "")]
#[cfg(target_pointer_width = "")]
```

### Compiler options

Target specific ABI.

```rust
#[cfg(target_env = "gnu")]
#[cfg(target_env = "msvc")]
#[cfg(target_env = "musl")]
#[cfg(target_arch = "mips")]
```

Moreover, you can customize dependencies as well.

```toml
[target.'cfg(windows)'.dependencies]
winrt = "0.7"

[target.'cfg(unix)'.dependencies]
nix = "0.17"
```

You can also add you own custom options by oassing `--cfg=myoption` to `rustc`.
Then inside code you can use `[cfg(myoption)]`.

### Versioning

Yeah nothing to say beside TAKE CARE OF IT. Stick to semantic versioning and
make sure you follow it. Also be careful about *Minimum Supported Rust Version*.

[Previous chapter - Error Handling in Rust](/posts/post-2022-08-25-error-handling-rust)

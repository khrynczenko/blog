+++
title = "Rust for Rustaceans #6: Macros"
date = 2022-10-07

+++

## Introduction

In this chapter we will go throug macros which are a tools for defining
transformation from one source code to another. Compiler replaces
macro invocation with the result of such transformation. Macros
allow **code generation**.

There are two types of macros:

- **declarative macros**,
- and **procedural macros**.

## Declarative Macros

Declarative macros are macros defined using `macro_rules!` syntax.

```rust
macro_rules! my_vec {
    () =>
    {
        {
            Vec::new()
        }
    };
    ($($x:expr),+ ) =>
    {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )+
            temp_vec
        }
    };
}
```

We invoce such macro like a normal function with `!` at the end.

```rust
fn main() {
    let v: Vec<usize> = my_vec![1, 2, 3];
    println!("{:?}", v);
}
```

Why rust macros are called *declarative*?
It refers to the fact that we do not specify how the transformation should
be done, we just say how the output should look like.

## When to use them

Their main use case is to replace coding the same part again and again.

Personally I used them when I had to define many parser function that had
only a small difference in the implementation and name.

```rust
macro_rules! token_parser {
    ($name:ident, $pattern:expr) => {
        #[allow(clippy::trivial_regex)]
        pub fn $name<'a>() -> impl Parser<'a, String> {
            make_token_parser(Regex::new($pattern).unwrap())
        }
    };
}

pub fn make_token_parser<'a>(pattern: Regex) -> impl Parser<'a, String> {
    let pattern_parser = cmb::regex(pattern);
    cmb::bind(pattern_parser, |value| {
        cmb::and(make_ignored_parser(), cmb::constant(value))
    })
}

token_parser! {make_bool_keyword_parser, "^boolean"}
token_parser! {make_number_keyword_parser, "^number"}
token_parser! {make_void_keyword_parser, "^void"}
token_parser! {make_array_keyword_parser, "^array"}
token_parser! {make_function_parser, "^function"}
token_parser! {make_if_parser, "^if"}
token_parser! {make_true_parser, "^true"}
token_parser! {make_false_parser, "^false"}
token_parser! {make_undefined_parser, "^undefined"}
token_parser! {make_null_parser, "^null"}
token_parser! {make_length_parser, "^length"}
token_parser! {make_else_parser, "^else"}
token_parser! {make_return_parser, "^return"}
token_parser! {make_while_parser, "^while"}
token_parser! {make_var_parser, "^var"}
token_parser! {make_assign_parser, "^="}
token_parser! {make_comma_parser, "^,"}
token_parser! {make_colon_parser, "^:"}
token_parser! {make_semicolon_parser, "^;"}
token_parser! {make_less_parser, "^<"}
token_parser! {make_greater_parser, "^>"}
token_parser! {make_left_paren_parser, r"^\("}
token_parser! {make_right_paren_parser, r"^\)"}
token_parser! {make_left_brace_parser, r"^\{"}
token_parser! {make_right_brace_parser, r"^\}"}
token_parser! {make_left_bracket_parser, r"^\["}
token_parser! {make_right_bracket_parser, r"^\]"}
token_parser! {make_not_parser, r"^!"}
token_parser! {make_equal_parser, r"^=="}
token_parser! {make_not_equal_parser, r"^!="}
token_parser! {make_plus_parser, r"^\+"}
token_parser! {make_minus_parser, r"^\-"}
token_parser! {make_star_parser, r"^\*"}
token_parser! {make_slash_parser, r"^/"}
token_parser! {make_id_string_parser, r"^[a-zA-Z_][a-zA-Z0-9_]*"}
```

**If your code changes based on type, use generics, otherwise, use macros.**

### How they work

Its nothing spectacular. Macro defines the output of macro invocation,
on the invocation rust compiler parses that output which is a series of tokens,
and places that at the place of invocation.

### How to Write Declarative Macros

Rust macros consists of two main parts: **matchers** and **transcribers**.
A given macro can have many matchers and each matcher has associated
transcriber.

```rust
macro_rules! /* macro_name */ {
    (/* 1st matcher */) -> { /* 1st transcriber */ };
    (/* 2nd matcher */) -> { /* 2nd transcriber */ };
}
```

The matcher below accepts a sequence ($()) of one more (+) comma-seprated (),)
key/value pairs given in key => value format.

`$($key:expr => $value:expr),+`

Here are *fragment* types:

- `:ident` for identifier,
- `:expr` for expression,
- `:ty` for types.

`$()` means to match pattern inside `()` repeatedly.

A transcriber uses *metavariables* defined in a matcher by prependning them
with `$`.

`$(map.insert($key, $value);)+`

Transcriber know which repetition to use by lookin at the meta variable used
inside anmd you are always force to use a metavariable when using repetition
with `$()`.

```rust

```

### Hygenic

Macros do not capture names defines outside of macro. Putting it explicitly
they are using lexical scoping.

## Procedural Macros

In procedural macros we actually can walk over the input token stream
and do antyhing we want with it in order to produce new token stream
for the output. This is what makes procedural macros so powerfull.

But this is hard so it is recommended to use `syn` crate to build
AST first and work on top of it inestad of just a series of tokens.

## Summary

Declarative macros are nice but they have restrictions. Using them to remove
repetitions can be useful. Procedural macros are more powefull because they
allow for defining the **procedure** transforming input tokens into
output tokens.

[Previous chapter - Testing](/posts/post-2022-09-22-rust-testing)


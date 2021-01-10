+++
title = "What is [[nodiscard]]?"
date = 2020-05-27

+++
## What is `[[nodiscard]]`?
`[[nodiscard]]` is another attribute that has been added into C++ in the
C++17 version. If you don't know what *attributes* are you can find the details
on the [reference page](https://en.cppreference.com/w/cpp/language/attributes)
but putting it simply they are things that
you can annotate types, functions and other things with and thus provide
some additional information to the compiler.

`[[nodiscard]]` can be applied, in particular to function, enum and
class declarations. For example they can appear in the following way.

```c++
class [[nodiscard]] NoDiscardClass {
};

enum class [[nodiscard]] NoDiscardEnum {
};

[[nodiscard]] bool is_discarded() {
}
```

## What does `[[nodiscard]]` do?
When it is used with a *class* or *enum* it indicates that when the value of
such type is returned from any function and not used in any way
the compiler should emit a warning to the user. Below is an example of such a
situation.

```c++
class [[nodiscard]] NoDiscardClass {
};

NoDiscardClass do_something() {
    return NoDiscardClass{};
}

int main()
{
    do_something();
    return 0;
}
```

We do not handle the value given by `do_something` but because we
annotated `NoDiscardClass` with `[[nodiscard]]` the compiler (in this case
MSVC) emits following warning.

**`warning C4834:  discarding return value of function with 'nodiscard' attribute`**

If we removed `[[nodiscard]]` this code would compile without any warnings.

In the event that `[[nodiscard]]` is used with function then it doesn't
matter whether the type it returns is annotated or not, the warning will be
emitted nonetheless. Again showing it in an example.

```c++
[[nodiscard]] bool is_positive(int number) {
    return number > 0;
}

int main()
{
    is_positive(10);
    return 0;
}
```

Again this code when compiled issues a warning that looks exactly the same as
the previous one.

## Why is it useful?
Being programmers and having to remember so many things when writing code,
it is easy to forget things. One of these things is to handle returned values.
I think that when function returns something it is not for no reason and it
is our job to do something with that value whether it is result of some
calculation or just error indicator.

Why would we call a function that returns a value, and then do nothing with
that value? This would probably mean that the function does something more,
like modify a state or perform *IO*. Silly example might look similar
to one below.

```c++
#include <filesystem>

[[nodiscard]] bool is_positive(int number) {
    return number > 0;
}

bool log_sqrt_to_file(const std::filesystem::path& filename, int number) {
    if (!is_positive(number)) {
        return false;
    }
    // logging part
}

int main(int argc, char* argv[])
{
    log_sqrt_to_file("log.txt", argc);
    return 0;
}
```

Here I can imagine that if we do not care whether logging with
`log_sqrt_to_file` succeeds we don't need to handle returned value.
By applying
`[[nodiscard]]` to the `log_sqrt_to_file` we would get that pesky warning
that we don't want because we discarded the value deliberately.
So how about adding `[[nodiscard]]` anyway and just assigning it
to some variable.

```c++
[[nodiscard] bool log_sqrt_to_file(const std::filesystem::path& filename, int number) {
    if (!is_positive(number)) {
        return false;
    }
    // logging part
}

int main(int argc, char* argv[])
{
    bool _ = log_sqrt_to_file("log.txt", argc);
    return 0;
}
```

Unfortunately this will give as another warning.  
`warning C4189:  '_': local variable is initialized but not referenced`

Fortunately there is a way
around it and that is coincidentally another attribute i.e. `[[maybe_unused]]`.
We can just add it to the assigned variable and voila.

```c++
int main(int argc, char* argv[])
{
    [[maybe_unused]] bool _ = log_sqrt_to_file("log.txt", argc);
    return 0;
}
```

We get the benefits of `[[nodiscard]]` and still maintain the flexibility.

Im am not so sure how `[[nodiscard]]` useful is with *classes* and *enums*
except maybe when those represent errors.

I hope that it has been useful quick overview of the `[[nodiscard]]`
attribute and I encourage you to think about using it in your own projects.

___
*Here comes my opinion so please bear with me...*  
I know there are cases
where we do not care about a returned value but I believe that those
are few and far between and because of that we should apply `[[nodiscard]]` to
all functions except when we have a good reason not to. I would love to have
it as default but yeah we are in C++ world and we have to consider things like
backward compatibility and impacting existing code-bases.

I must say that I have a small beef with attributes in general. I feel
that they make C++ code, which is already very verbose, even more cluttered.
We already have `constexpr`, `noexcept`, `inline`, `virtual`, `override` etc.
Things are
are getting out of hand. Nowadays declarations are starting to look like little
monsters.
I pity those who start their journey with C++ and have to look at modern C++.
There is just so much cognitive overhead when compared to *"C with classes"* or
other languages.

```c++
template<typename T, typename = std::enable_if<std::is_arithmetic<T>::value>>
[[nodiscard]] constexpr bool is_positive(T&& number) noexcept;
```

So it is not a problem specific to attributes per se. Fixing things by just
adding stuff leads to such situations.

I started learning when C++11 was becoming a thing and learned C++ gradually.
I fear that nowadays newcomers can become overwhelmed and hence discouraged
to use C++.

We get all those goodies in modern C++ (which I love and encourage others
to use) but there is some cost. I think newcomers can find it harder and harder
getting into the language and that worries me.

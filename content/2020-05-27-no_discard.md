+++
title = "What is [[nodiscard]] and why it is usefull."
date = 2020-05-27

[taxonomies]
categories = ["C++"]
tags = ["C++", "C++17", "C++ attributes"]
+++

# What is `[[nodiscard]]`?
`[[nodiscard]]` is another attribute that has been added into C++ in the
C++17 version. If you don't know what *attributes* are you can find the details on the [reference page](https://en.cppreference.com/w/cpp/language/attributes/nodiscard) but putting it simply they are things that
you can annotate types, functions and other things with and thus provide
some additional information to the compiler.

`[[nodiscard]]` particularly can be applied in function, enum and
class declarations. So for example they can appear in the following way.

```c++
class [[nodiscard]] NoDiscardClass {
};

enum class [[nodiscard]] NoDiscardEnum {
};

[[nodiscard]] bool is_discarded() {
}

```

# What does `[[nodiscard]]` do?
In case where it is used with a *class* or *enum* it indicates that when value of such type is returned from any function and not used in any way
a compiler should emit warning to the user. Below is an example of such
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
So we do not handle value given by `do_something` but because we
annotated `NoDiscardClass` with `[[nodiscard]]` compiler (in this case MSVC)
emits following warning.

**`warning C4834:  discarding return value of function with 'nodiscard' attribute`**

If we would remove `[[nodiscard]]` this code would compile without any warnings.

In case when `[[nodiscard]]` is used with function then it doesn't
matter whether the type returned by it is annotated or not, warning will be
emitted nonetheless. So again showing it in an example.

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

Again this code when compiled issues warning that looks exactly the same as
the previous one.

# Why is it useful?
Being programmers and having to remember so many things when writing code
it is easy to forget things. One of such things is to handle returned values.
I think that when function returns something it is not for no reason and it
is our job to do something with that value whether it is result of some
calculation or just error indicator.

Why would we call a function that returns a value and then do nothing with
it? This would probably mean that the function does something more like modifies a state or performs *IO*.

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

Here I could imagine if we do not care whether logging with `log_sqrt_to_file` succeeded we don't need to handle returned value. If we would apply
`[[nodiscard]]` to the `log_sqrt_to_file` we would get that pesky warning
that we don't want in this case because we discard the value purposefully.
So how about adding [[nodiscard]] and just assigning it to some variable.

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
around it an that is coincidentally another attribute i.e. [[maybe_unused]].
So we can just add it to the assigned variable and voila.
```c++
int main(int argc, char* argv[])
{
	[[maybe_unused]] bool _ = log_sqrt_to_file("log.txt", argc);
	return 0;
}
```

So we get the good of `[[nodiscard]]` and still have the flexibility.

Im am not so sure about usefulness of `[[nodiscard]]` on *classes* and *enums*
except when they the indicate errors maybe.

I hope that this is useful overview of the `[[nodiscard]]` attribute and
I encourage you to think about using it in your own projects.

___
*Here comes my opinion so please bare with me...*  
So I know there are cases
where we do not care about returned value but I believe that those
are few and far between and because of that we should apply `[[nodiscard]]` to
all functions except when we have a good reason not to. I would love to have
it by default but yeah we are in C++ world and we have to consider things like
backward compatibility and impacting existing code-bases.

Albeit I have beef with attributes in general is that it makes C++ code which is already very verbose even more cluttered with keywords. We already have
`consextpr`, `noexcept`, `inline`, `virtual`, `override` etc. Things are
are getting out of hand. Nowadays declarations are starting to look like monsters. I pity those who start their journey with C++ and have to look
at modern C++. There is just so much cognitive overhead when compare to `C with classes` or other languages.

```c++
template<typename T, typename = std::enable_if<std::is_arithmetic<T>::value>>
[[nodiscard]] constexpr bool is_positive(T&& number) noexcept;
```
So I stared learning when C++11 was becoming a thing and learned C++ gradually.
I fear that nowadays newcomers can become overwhelmed and hence discouraged
to use C++.

Yeah we get all these goodies in Modern C++ but there is some cost. I think getting into the language for newcomers might be becoming harder and harder.





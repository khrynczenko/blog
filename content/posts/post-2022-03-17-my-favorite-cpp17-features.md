+++
title = "My favorite C++17 features"
date = 2022-03-17

+++

## C++17 standard

The seventieth standard of C++ is more similar to C++11 than C++14 in terms
of the size of the scope of the changes. A lot of features have been added
to both the language and the standard.

Once again I am going to list only a few of them, a few that I like the most.

## Language features

### Structured bindings

Structured bindings make it easier to extract internal structure data and bind
it to seperate names in one step.

Pre C++17:

```c++
std::pair<int, int> point{1, 2};
int x = point.first;
int y = point.second;
```

Post C++17:

```c++
std::pair<int, int> point{1, 2};
auto [x, y] = point;
```

This can be used with many other classes like `std::tuple`, `std::array` and
even user-defined structures.

### if/switch with initializer

This is something I longed for a long time. Finally we cant have values
that do not leak out of the scope of the conditional.

Pre C++17:

```c++
const auto vehicle = f();
if (!vehicle.has_gas()) {
    vehicle.fill_tank();
}

// `vehicle` is still in the symbol table
```

Post C++17:

```c++
if (const auto vehicle = f(); vehicle.has_gas()) {
    vehicle.fill_tank();
}
```

### New attributes

C++17 added three new attributes that I found very useful. Namely
`[[nodiscard]]`, `[[fallthrough]]`, and `[[maybe_unused]]`. From this bunch
I like the `[[nodiscard]]` the most because it is another tool for combating
common mistakes.

`[[nodiscard]]` casues compiler to emit a warning when you do not handle
a value returned by a function.

```c++
[[nodiscard]] bool is_positive(int number) {
    return number > 0;
}
```

To be honest I have some issues with attributes as well because I think they
add some cognitive overhead to the language that is already burdened with it.
Things like these should be enabled by default IMHO but it is what it is
and it is better than having nothing at all.

I wrote an article specificly about `[[nodiscard]]` that you can find
[here](https://www.blog.khrynczenko.com/posts/post-2020-05-27-nodiscard/).

## STL features

Features that came to the Standard Template Library in C++17 are very useful.
Among these I am going to mention, `std::byte`, `std::filesystem`, and
*parallel algorithms* are great additions.

### `std::variant`

This class represent a type-safe union and being type-safe is always welcomed.

```c++
#include <variant>

std::variant<int, std::string> id{ 1 };
std::get<int>(v); // == 12
std::getif<std::string>(&v); // returns nullptr
std::get<1>(v); // throws an error
```

### `std::any`

This is just another type-safe wrapper aroung values of any type. Everything
is better that `void *`.

```c++
#include <any>
#include <string>

std::any id {5};
id.has_value() // == true
int id_copy = std::any_cast<int>(id);
id = std::string{"stringid"};
std:string id_copy2 = std::any_cast<std::string>(id);
```

### `std::string_view`

`std::string_view` is just a non-owning reference to a string. What is cool
about it is that when using `std::string_view` instead of `const std::string&`
you drop one level of indirection (you basically follow only one pointer
instead of two). I like it because it also provides more meaning to the
context.

```c++
const std:string name = "Foo Bar";
const std::string_view cppstr{name};
```

### `std::optional`

Finally, we get a type that can be used to indicate a possibility of error
(or rather a missing value) in the API. It can the replace use of exceptions,
with the advantage of being type-safe. It also can replace these pesky
functions that use `bool` as a return value to indicate success/failure and
then an output parameter for the actual return value (which should be avoided,
more on that topic
[here](https://www.blog.khrynczenko.com/posts/post-2021-09-24-output-parameters/)
and [here](https://www.blog.khrynczenko.com/posts/post-2021-02-16-taking-advantage-of-the-type-system-in-c-plus-plus/)).

```c++
struct FirstName {
    static std::optional<FirstName> Create(std::string name) {
        if (!is_name_valid(name)) {
            return std::nullopt;
        }
        return FirstName(name);
    }
    const std::string value;
private:
    FirstName() = delete;
    FirstName(std::string name) : value(name) {}
};
```

## C++17 is a great addition

I feel like C++17 provides a lot of useful features that I am and will happily
use. It shows a general direction, and care for providing programmers
with the tools they need. What are your favorite features? :)

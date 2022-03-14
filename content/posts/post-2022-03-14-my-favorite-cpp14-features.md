+++
title = "My favorite C++14 features"
date = 2022-03-14

+++

Following my previous post here are my favorite C++14 features. There are
not that many because C++14 in itself was a rather small update to the
of the standard.

## Language updates

There were not many language updates. The more noticable ones are *generic
lambdas*, *type deduction for function return types*, *variable templates*,
and *looser restricitons when using `constexpr`*.

From these, for me the improved `constexpr` experience is the biggest one. Being
able to use `for` loops, `if`s and `switches` makes `constexpr` much more
useful tool.

```c++
constexpr bool are_positive(int values[], int size) {
    for (int i = 0; i < size; ++i) {
        if (values[i] < 1) {
            return false; 
        }
    }

    return true;
}
```

When it comes to smaller features I really like that we can now use binary
literals and digit separators.

```c++
int value = 0b101010; // yay
int big_number = 1'123'123'123 // yay2
```

Binary numbers are especially handy. I know most of the time hexadecimal
values are a better fit but I came into problems where binary made more
sense due to the context.

## STL updates

There were not many new things that came to the STL but a lot of already
existing things were extended with `constexpr` which is super cool IMHO.
Besides that `std::make_unique` got added which I used always when making
smart pointers. It is nice because it will create value in place and is
just more concise.

```c++
#include <memory>

struct Point {
    Point(int x, int y) : x(x), y(y) {
    }
    int x, y;
};

int main() {
    auto p1 = std::make_unique<Point>(1, 2);
    auto p2 = std::unique_ptr<Point>(new Point(1, 2));
}
```

## C++14

In general, there isn't much in C++14 besides small improvements here and there.
The focus on taking advantage of `constexpr` has been nice, and the few
quality of life improvements like binary literals and digit separators are
always welcome I think.

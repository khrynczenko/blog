+++
title = "My favorite C++11 features"
date = 2022-03-11

+++

Here are some of my favorite features that came with the C++11 standard. There
are many more and I am not saying other are less useful. It is just that these
below are the ones I personally find most valuable.

## Language updates

### Lambda Expressions aka Anonymous Functions aka Closures

Functional programming paradigm has been slowly getting incorporated into prety
much every mainstream language even to those that are considered
to be object-oriented focuses. You could argure that closures are not
specific to functional programming but the fact is that they are a fundamental
feature in all functional programming languages.

I am not going to explain what they are here. I am just going to say that I love
how convenients they are. They are usfeull in algorithms, so you don't need
to define functors anymore which are more verbose.

```c++
#include <algorithms>
#include <vector>

int main() {
    const std::vector<int> numbers = {-2, -1, 1, 2, 3, 4};
    int positive_count =
        std::count(std::begin(numbers), std::end(numbers), [](const int number) {
            return number > 0;
        });
}
```

This is quick and easy and doesn't require defining unnecessary class.

Closures are also very nice when you are interacting with some old API
that uses output parameters, which would normally block you from marking your
value as `const`.

```c++
void choose_random(int& x);

int main() {
    const int x = []() {
        int v;
        choose_random(v);
        return v;
    }();
    // yay x is `const`
}
```

### The `nullptr` keyword

Oh finally we can get rid of this error-prone `NULL` all over the codebase.
`NULL` is bad because it not strongly typed like `nullptr` is. Consider
the below example.

```c++
int perform(const char* x);
int perform(int x);

int main() {
    perform(NULL); // oops
    perform(nullptr); // alright
}

```

More examples could be made. Just use `nullptr` when working with pointer
types.

### The `auto` keyword

This is another life improver.

```c++
#include <vector>

int main() {
    const std::vector<int> numbers = {-2, -1, 1, 2, 3, 4};
    std::vector<int>::const_iterator no_auto= numbers.cbegin();
    auto with_auto = numbers.cbegin();
}
```

Not much to add here.

## The STL

### Smart pointers

Raw pointers should be avoided, they are confusing because it is rarely
obvious who is responsible for releasing the memory, who takes ownership etc.

`std::unique_ptr` and `std::shared_ptr` and `std::weak_ptr` are the way to go.

There is not much to add here.

### Thread library

Thread library is great. It is cross-platform, easy to use and for simple tasks
where you just need to spin another thread do some work you need nothing else.

```c++
#include <thread>
#include <iostream>

void hello() {
    std::cout << "Hello";
}

void world() {
    std::cout << " world!";
}

int main() {
    std::thread t1(hello);
    t1.join();
    std::thread t2(hello);
    t2.join();
}
```

## C++11

There is tones more. But these in particular I find using a lot and I think
they added a lot of value to the language.


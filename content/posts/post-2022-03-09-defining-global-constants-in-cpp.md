+++
title = "Defining global constants in C++"
date = 2022-03-08

+++

## Global constants in C++

Last two years I have been cycling between different languages and technologies
more than I am used to. Because of that, I spent less time using C++ than I
would like to. But I can say one thing. Whenever I come back I need to relearn
how to properly define a global constant. I know it sounds ridiculous. But I
swear in modern C++ there are a lot of ways you can do that, each having some
advantages and disadvantages. So let's dive in and go over how we can create
global constants in C++.

## Defining constants in header files

Let's start with the usual approach used before `constexpr` became a thing.
So we are going to use regular `const` keyword and define some values in the
header file.

```c++
// constants.hpp

const double PI = 3.14;
const double PHI = 1.61;

```

Now we could include this header in any source file and use these constants
as we please.

```c++
// main.cpp
#include <iostream>
#include "constants.hpp"

int main() {
    std::cout << PHI * PI;
    return 0;
}

```

It is important to notice that these values `PI` and `PHI` have *internal*
linkage. It means that *each* translation will use its own copy of these
values. In other words, each object file corresponding to each cpp file will
have its own `PI` and `PHI` definitions. On the other hand, this approach is
super simple and it works.

Personally, I don't think it is too much of a problem. I would not consider
such a thing to be code bloat. But hey I usually don't care for the size
of the produced binary but it might depending on the domain you are coming
from.

The bigger problem I can see is that any change to values defined in the
header file might cause the recompilation of all the files that include the
header.
If the header is included on mass and the values will get change a lot
(for example to adjust them during prototyping), this could cause a headache.

## Defining externally linked constants in source files

We can remove the bloat and enforce the final output to use one value (per
constant) to be shared across different translation units. We do that by
defining these values in a source file (.cpp) instead of the header file.

```c++
// constants.cpp

extern const double PI = 3.14;
extern const double PHI = 1.61;

```

In order for these values to have *external* linkage, we have to add the
`extern` keyword because `const` value as mentioned earlier has an *internal*
linkage by default.

```c++
// constants.hpp

extern const double PI;
extern const double PHI;

```

Now we can just put forward declarations in the header file and voila. We now
have these constants linked and shared across all translation units instead
of them being defined in each separately. So this approach removed all the
problems that were caused by defining the values in the header file, i.e.,
the code bloat and the unnecessary recompilation.

## Defining constants using constexpr

In the newer versions of C++ we can use `constexpr`. `constexr` is
great and when something is known at compile time there is no reason not
to use it instead of `const`.

Unfortunately, you cannot define `constexpr` values inside a source file (.cpp
file), and they provide a forward declaration for them in the header file. This
is because `constexpr` values must be known at compile-time and when you would
include a header file with a forward declaration for some `constexpr` value
you don't know the actual value. It is somewhere but to that translation unit,
it is not visible just yet.

So once again we need to define the values inside the header file.

```c++
// constants.hpp

constexpr double PI = 3.14;
constexpr double PHI = 1.61;

```

But why would we use `constexpr` anyway? What is the benefit? The benefit is
that we can use these values in `constexpr` contexts. Unfortunately by defining
these in the header file once again we are causing unnecessary
code bloat and recompilation.

## Defining inline constexpr values

Finally, we come to the final form of how we can define global constants.
`inline` come to rescue us from this pesky code bloat that I mentioned so many
times. You see in **C++17** the `inline` keyword can be used with variables. And
it gives two properties to such variables:

- there may be more than one definition of such values (as long as these
definitions are identical),
- these values will have the same address in every translations unit,

which basically means that the values will be shared across all the
translations units instead of being defined in each. This is what we wanted.
We end up with the following.

```c++
// constants.hpp

inline constexpr double PI = 3.14;
inline constexpr double PHI = 1.61;

```

## So many constants

I think the `inline constexpr` approach is the one to be used as a rule of
thumb. It has the advantage of values being `constexpr` and being defined
once for every translation unit. There are some other intricacies to defining
`constexpr` values, especially when doing it inside functions. Then `static`
has an actual impact but honestly, from what I have read I have no idea which
approach is best there.

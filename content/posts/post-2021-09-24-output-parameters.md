+++
title = "Revisiting output parameters usefulness (in C++)"
date = 2021-09-24

+++

> This article was originally written as a blog post for the
> [**Bulldogjob**](https://bulldogjob.pl)
> blog. This is the second time we have done this and again I am thankful to both
> **Bulldogjob** and my company **Rockwell
> Automation** for the opportunity to work on it and publishing it.
> You can find the original version
> [here](https://bulldogjob.com/articles/1323-revisiting-output-parameters-usefulness-in-c).

Output parameters is a concept that was quite alien to me until I have started
working on legacy software. It is not that I have never seen output parameters.
I have but it was a rare occurrence. I was already writing C++ for several
years, both professionally and as a hobby, and I was never properly introduced
to this concept. You can imagine my confusion when I started working on a
codebase where a bigger part of functions is using output parameters. So,
what are output parameters, and why I am advocating against their liberal use?
I will try to explain that in the following sections.

## Output parameters

Nowadays, I have been interacting with a codebase where a significant number of
functions give their results through parameters instead of using the return
value. That is what output parameters are, i.e., parameters that are used for
providing results. Below is the example of a function that creates a sequence
of powers of two using output parameters.

```c++
void powers_of_two(const size_t n, std::vector<size_t>& powers) {
  powers.reserve(n);
  for (size_t i = 0; i < n; ++i) {
    powers.push_back(std::pow(2, i));
  }
}
```

On the opposite, below is an example of the function that provides the result
through the return value.

```c++
std::vector<size_t> powers_of_two_using_return_value(const size_t n) {
  std::vector<size_t> powers;
  powers.reserve(n);
  for (size_t i = 0; i < n; ++i) {
    powers.push_back(std::pow(2, i));
  }
  return powers;
}
```

The difference might seem subtle but there are a lot of nuances involved.

As a side note, I know that these functions could be implemented way better for
example using algorithms from the `<algorithm>` header but this is not a focus
of this article.

So, what do you think? Which function is better in your opinion, what are the
advantages/disadvantages of either approach? I will already uncover that the
advantage of output parameters is that they are more efficient, at least that
is the reason people state most often when asked about it. This actually is
not entirely true. But before diving into that let’s go over more obvious
disadvantages of output parameters.

## Output parameters might be confusing

This argument is perhaps the most subjective one. Yes, in my opinion, output
parameters are confusing. It is I believe common understanding that a function
takes arguments and produces results. Arguments are the function’s input, not
its output. Let’s look at the signatures of both functions.

```c++
void powers_of_two(const size_t n, std::vector<size_t>& powers);
 
std::vector<size_t> powers_of_two(const size_t n);
```

While this simple example might not be convincing, I think that when people are
using some API they are most interested in what a function produces first
(because that is their true desire), then they look at the required parameters.
They want to obtain some value and for that, they look first at the return type.
Let’s look at a more complicated function signature.

```c++
void parse_dimension_request(const std::vector<char>& bytes,
                             bool big_endian,
                             size_t& x, size_t& y, size_t& z);
```

I hope you can notice that in order to find out what a given function returns
we need to look over all the parameters and separate them into input/output parameters.

In essence when output parameters are involved, I must perform two steps in
order to find out what is a result of a function:

1. Look at the return type, notice there is a void type (or sometimes bool,
error code, etc.).
2. Go over the parameters and figure out which ones are used for the results.

With normal functions, I just perform step one. Moreover, assuming that a
parameter is an output parameter solely on the fact that it is passed by
reference might also be not correct.

Okay with this out of the way let’s go to things that are a little bit more interesting.

## Output parameters make const initialization harder

I think that most people agree that `const` keyword should be used liberally in
our codebases. It prevents errors, helps the compiler with optimizations, and
makes code easier to reason about. Item 3 in Scott Meyer’s “Effective C++:
55 Specific Ways to Improve Your Programs and Designs” is titled “Use const
whenever possible”. Here is the quote of the last paragraph in this chapter:
“As I noted at the beginning of this Item, `const` is a wonderful thing. On
pointers and iterators; on the objects referred to by pointers, iterators, and
references; on function parameters and return types; on local variables; and
on member functions, `const` is a powerful ally. Use it whenever you can. You'll
be glad you did.”

The problem is that output parameters discourage usage of `const` keyword. Let’s
try to use the `powers_of_two` function, first without the output parameter.

```c++
void some_function()
{
  // ...
  const std::vector<size_t> powers = powers_of_two(3);
 
  //...
}
```

No problems here. We `const` initialize `powers` by calling `powers_of_two`
directly. Now let’s look at the output parameter variant.

```c++
void some_function()
{
  // ...
  const std::vector<size_t> powers;
  powers_of_two(3, powers); // error, we are trying to modify `const` value
 
  //...
}
```

This code does not compile. We cannot mark `powers` with `const` if we want to
use output parameter variant of `powers_of_two`. We need to drop the `const`
modifier.

```c++
void some_function()
{
  // ...
  std::vector<size_t> powers;
  powers_of_two(3, powers);
 
  //...
}
```

This works but we lost all the perks of using `const`. Imagine that every
function in a codebase uses output parameters. That would mean losing the
ability to mark values `const` entirely.

There is a way around that using a lambda function and calling it at the point
of definition.

```c++
const std::vector<size_t> powers = []() {
    std::vector<size_t> powers;
    powers_of_two(3, powers);
    return powers;
  }();
```

This, kind of helps us solve this issue. The inner powers value is still not
marked as `const` but when it is confined in such a little scope that is okay.
The problem with this approach is that it adds a lot of verbosity to our code.
It is okay to do this when you must. It is not okay when you must do this
everywhere.

## Output parameters are not more efficient

And finally, we come to the one reason that is most often brought when
discussing the usefulness of using output parameters. The reason is that
functions that use output parameters do not incur the penalty of unnecessary
copying. This copy ought to occur when returning from a function. Whether this
is true or not depends mostly on the compiler/standard you are using. The fact
is that every modern compiler performs **return value optimization (RVO)**. It
means that the compiler can optimize away the copy operation. Moreover, modern
compilers can perform **named return value optimization (NRVO)**.

How RVO and NRVO works? Another article could be written about this but in
essence, the compiler is able to avoid constructing temporary objects by
directly initializing value with the arguments that were computed in the
function. Let’s start with a simple example.

```c++
struct Point {
  int x;
  int y;
};
 
Point add_points(const Point& lhs, const Point& rhs) {
  int new_x = lhs.x + rhs.x;
  int new_y = lhs.y + rhs.y;
  return Point{ new_x, new_y };
}
 
int main() {
  const auto p1 = Point {1, 2};
  const auto p2 = Point {3, 4};
  const auto p3 = add_points(p1, p2);
}
```

So, base on the previous information `p3` is going to be directly initialized.
Therefore, the code would translate to something different.

```c++
int main() {
  const auto p1 = Point {1, 2};
  const auto p2 = Point {3, 4};
  int new_x, new_y = add_points(p1, p2); 
  // ^^^ this is what it gets down to behind the scene
  const auto p3 = Point(new_x, new_y);
}
```

Keep in mind this is more of a pseudocode, but it illustrates the idea.
Is this something that we can rely on? I think yes. [There are articles going
back to Visual C++ 2005 explaining showcasing these optimizations.](
https://docs.microsoft.com/en-us/previous-versions/ms364057(v=vs.80))
If that’s not convincing maybe the C++ standard is. Since C++17, the standard
mandates guaranteed copy elision.

Going back to code efficiency, Chandler Carruth known for speaking about code
performance and efficiency, when asked about improving efficiency using output
parameter in one of his examples answered (edited just for clarity)
“… please don’t. That’s bad. Why is that bad though? You can pass a pointer in.
But when you do, all of a sudden, there is no way to tell where that pointer
came from. There is no way to tell whether that pointer points to the same
thing that stuff going into X’s constructor points to. We lose a ton of
information about what the semantics of this program are. And if you actually
have a very nice optimizing compiler it loses that information as well. And you
will see it actually produce poorer code. Output parameters are not faster.
It is kind of a myth. RVO and NRVO and move semantics essentially remove need
for output parameters from the performance stance. There are extreme edge cases…”.

In short, output parameters can sometimes even hurt the performance because
they interfere with compiler’s assumptions regarding our code. I highly
recommend watching Chandler Carruth’s talks on YouTube.

## Some caveats

The previously mentioned optimizations are not always possible, and there
are situations where the compiler will not be able to perform copy elision.
Unfortunately, the rules of RVO and NRVO can be quite convoluted especially
when edge cases are considered. As a rule of thumb if your function is
not overly complicated you can expect copy elision to kick in. If the part
of the code you’re working on is performance-critical then you should probably
benchmark it anyway and based on that make a decision on whether to use output
parameters or not.

[To get an overview of how RVO and NVO works I recommend the following
article.](https://www.fluentcpp.com/2016/11/28/return-value-optimizations/)

I could also imagine that sometimes output parameters might be a better
fit for some API design and I do not question such situations even though I
cannot come up with an example.

## Summary

In this article, I am not trying to dismiss the usage of output parameters
completely. I understand C++ is a widely used language, there are many domains,
many different industries and my view of the situation might be limited. What I
am trying to suggest is that if you are working on a C++ codebase and not using
some arcane compiler, it might be worth considering using return values for the
results as a default approach and fall back to output parameters if necessary.

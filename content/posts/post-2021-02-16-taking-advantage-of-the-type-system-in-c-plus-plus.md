+++
title = "Taking advantage of the type system (in C++)"
date = 2021-02-16

+++

> This article was originally written as a blog post for the
> [**Bulldogjob**](https://bulldogjob.pl)
> blog. I am thankful to both **Bulldogjob** and my company **Rockwell
> Automation** for the opportunity to work on it and publishing it.
> You can find the original version
> [here](https://bulldogjob.pl/articles/1265-taking-advantage-of-the-type-system-in-c).

## Introduction

After working on several projects now I believe that people do not use
facilities that come with using compiled programming languages that have
capable type systems such as C++, C, Java etc. I found that programmers often
too much rely on the primitive types and run-time checking which often leads to
bugs that are often hard to detect. In this article I would like to present
simple techniques to make your code less error-prone, more readable, and
maintainable, simply by using types.

## Example

Let us start with an example of a code that we will improve step by step
throughout the article. Assume that we are working on a shipping/ordering
software for a coal selling company. Inside there is a system that is
responsible for initiating a shipping process with a given amount of coal
to a customer.

```C++
struct Address {
    Address() = delete;
    Address(std::string street,
        std::string city,
        std::string zip_code,
        std::string country);

    const std::string street;
    const std::string city;
    const std::string zip_code;
    const std::string country;
};
 
struct Customer {
    Customer() = delete;
    Customer(std::string first_name, std::string last_name, Address address);
    const std::string first_name;
    const std::string last_name;
    const Address address;
}; 
 
void ship_order(Customer customer, std::uint16_t amount) {
    // initiate shipping
}

```

We can see that we have a `ship_order` function that is responsible for the
shipping. It takes two parameters, one of which is a `Customer` structure
with all the required details related to the customer. The second parameter is
related to the amount of coal that should be delivered.

## Simple types

Let us focus on the first function parameter – the `customer` parameter. If we
look at the structure definition a couple of things should raise your
awareness.  The first and the last name of the customer are defined as strings.
But is it correct? The type tells us that for example the name of the user can
be “Android-321#@!”. This is a valid string, unfortunately this is not a valid
name. This is an example of a simple type, a type which is represented by a
primitive type, but it is not exactly the same. One could argue that these
values for the first name and the last name could be checked inside the
constructor like below.

```c++
Customer(std::string first_name, std::string last_name, Address address)
    : first_name(first_name)
    , last_name(last_name)
    , address(address) {
    if (!is_name_valid(first_name) || !is_name_valid(last_name)) {
        throw std::invalid_argument("Invalid name");
    }
}
```

While this might look good at first glance, this approach is severely lacking.
Firstly, this requires us to dive into implementation of the `Customer`
structure to be sure that the names are validated. Secondly this approach does
not scale well. If there are other places where we would like to represent
names without the `Customer` structure we will need to again make sure that
those are valid. This puts a lot of responsibility to keep the integrity of the
system on our shoulders. We start to rely on the fact that nobody has made a
mistake, and all the logic that touches such variables will always be correct.
In a small codebase this all might seem to be okay and trackable, but I hope
you can imagine that when the code grows this might get problematic
(from my experience it always does). I hope you see now that we can do better.

## Maintaining integrity with simple types

We are now going to tackle the problem of keeping the integrity within our
system throughout its lifetime. We start from introducing a new type which
will take role of representing names.

```c++
struct Name {
    Name() = delete;
    explicit Name(std::string name);
    const std::string value;
};
```

Now we replace the old string types from the `Customer` type with the `Name`
type.

```c++
struct Customer {
    Customer() = delete;
    Customer(Name first_name, Name last_name, Address address)
        : first_name(first_name)
        , last_name(last_name)
        , address(address) {
    }
    const Name first_name;
    const Name last_name;
   const Address address;
};
```

Just by doing that we now do not mislead people into believing that the name
variables can hold anything that string can (not entirely, but more on that
later). Moreover, as long as the value of the type `Name` can only be built
with valid string we now can be sure that where a value of the type `Name` is
present, there is no need to check whether it is valid.

The validation could look similarly to what we had present in the `Customer`
structure before.

```c++
struct Name {
    Name() = delete;
    explicit Name(std::string name) : value(name) {
        if (!is_name_valid(name)) {
            throw std::invalid_argument("Invalid name.");
        }
    }
    const std::string value;
};
```

What if the requirements for a valid name differ between the first name and the
last name? For example a first name cannot contain the hyphen (-) sign but a
last name can (two part last names are completely valid and not that uncommon).
We can just create two independent types that apply their own business rules.

```c++
struct FirstName {
    FirstName() = delete;
    explicit FirstName(std::string name);
    const std::string value;
};
 
struct LastName {
    LastName() = delete;
    explicit LastName(std::string name);
  const std::string value;
};
 
struct Customer {
    Customer() = delete;
    Customer(FirstName first_name, LastName last_name, Address address)
        : first_name(first_name)
        , last_name(last_name)
        , address(address) {
    }
    const FirstName first_name;
    const LastName last_name;
    const Address address;
};
```

Personally, even if the rules were the same, I would still make two separate
types for them to keep them distinct. This would enforce the compiler to issue
an error if someone tried passing a first name argument to the function that
wanted last name. Some people might find it a little of an overkill, I do not.
I believe in taking advantage and removing possibility for error where
possible, but of course all use cases vary, and you should use your own
judgment.

We should not forget about our `Address` structure. Obviously addresses, zip
codes, and countries cannot hold just any string. I believe that you can figure
it out yourself now how we apply the same technique there. Now our `Customer`
and `Address` structure should look something like below.

```c++
struct Address {
    Address() = delete;
    Address(Street street, City city, ZipCode zip_code, Country country);
    const Street street;
    const City city;
    const ZipCode zip_code;
    const Country country;
};
 
struct Customer {
    Customer() = delete;
    Customer(FirstName first_name, LastName last_name, Address address);
    const FirstName first_name;
    const LastName last_name;
    const Address address;
};
 
void ship_order(Customer customer, std::uint16_t amount) {
    // initiate shipping
}
```

This code looks much less worrisome now. Additionally, we restricted validation
code only to the constructors for the simple types. No more relying on yourself
and other programmers to keep all the values proper and validating them all
over the place. Yet there is still some room for improvement.

## Representing units of measure

Now let us take care of the second parameter of the `ship_order` function. This
parameter corresponds to the amount of coal to be shipped to a customer.
Its type is a 16-bit unsigned integer. Looking at the interface of a function
we cannot really know what units of weight it uses and if there are any
restrictions on a shipped amount. From the requirements we obtained from the
client we gathered that we cannot ship more than 10000 kg of coal. That is
probably why 16-bit unsigned integer was used, since it is the smallest integer
type that can hold value of 10000. If we were a little bit luckier someone
would add the name of the unit to the parameter name like so.

```c++
void ship_order(Customer customer, std::uint16_t amount_kg);
```

While this somewhat improves things and makes it more obvious someone can still
make a simple mistake and pass a value that does not correspond to kilograms.

```c++
void resolve_order(/* ... */) {
    // ..
    std::uint16_t tons_to_send = parse_amount(form);
    ship_order(customer, tons_to_send);
}
```

This code would compile without emitting any error, not even a warning. This is
a type of a problem that is hard to detect before it shows in the production,
and even then, finding it is a tedious process. Any compiler will help us with
such problems if we just give him a little more concrete information.

```c++
enum class WeightUnit {
    MG,
    G,
    KG,
    T,
};
 
template<WeightUnit>
struct Weight {
public:
    explicit Weight(std::size_t amount) : amount(amount) {
    }
    const std::size_t amount;
};
 
void ship_order(Customer customer, Weight<WeightUnit::KG> amount) {
    // initiate shipping
}
```

We introduced a new enumerator that describes what unit of weight we are using.
Moreover, we introduced a new weight type that stores the amount under a given
unit. It has a template parameter that corresponds to the weight unit and that
must be provided at compile time. Now if we would try to do the same as in the
example before the compiler would gracefully emit an error and save us from
a lot of trouble.

```c++
void resolve_order(/* ... */) {
    // ..
    Weight<WeightUnit::T> tons_to_send = parse_amount(form);
    ship_order(customer, tons_to_send); // This no longer compiles!
}
```

This will no longer compile thus providing the type-safety we want (the kg unit
is expected). This is a rather bare-bones approach to this problem, but it
shows once again that we can use type-system to our advantage. I specifically
omitted any details regarding conversion between different weight units to not
cloud the general idea and keep the article concise. There are many ways to
tackle this problem and there are some battle tested libraries that do that,
you don’t have to solve it yourself (though it is not that hard). There is a
*Boost Units* library or the work in progress *units* library that can help you
with the problem of working with units.

As for the second requirement that the shipped amount should not exceed 10000
kg I think we already know how to approach it now. We introduce a new simple
type of course.

```c++
struct ShippedAmount {
    explicit ShippedAmount(Weight<WeightUnit::KG> amount) : amount(amount) {
        if(amount > Weight<WeightUnit::KG>(10000)) {
            throw std::invalid_argument("Shipping amount cannot exceed 10'000 kilograms.");
        }
    }
    const Weight<WeightUnit::KG> amount;
};
 
void ship_order(Customer customer, ShippedAmount amount) {
    // initiate shipping
}
```

Now, if we need to use values that are shipping amounts in other places in the
code, the validation stays in one place only and we can be sure that everything
stays correct.

## Documenting effects and working with errors

To that point I omitted one problem that kept one showing in several places.
That is the problem of validation in constructors that is not in any way
showcased by the interface of those constructors. Below is the `FirstName`
class that was shown earlier.

```c++
struct FirstName {
    FirstName() = delete;
    explicit FirstName(std::string name);
    const std::string value;
};
```

One small problem is that we use an exception for something that really is not
an exception, and we force on the user of the class to perform exception
handling which has its own issues. More importantly the signature of the
constructor does not say much about that it performs any kind of validation.
How is a user supposed to know that he has to handle possible exceptions?
One way of approaching that would be to introduce exception list like below.

```c++
explicit FirstName(std::string name) throw(std::invalid_argument);
```

This at least somehow tells the reader that this function might throw,
which would indicate some kind of validation, especially if we would create
our own exception type with a better name. Unfortunately, we still use
exceptions, so it solved only part of the problem. Moreover, exceptions
specifications have been deprecated since C++11 and removed in C++17. There
must be another way.

Of course, once again we can use types. For errors specifically C++ introduces
new std::optional type (since C++17). This type is a representation for a value
that might be present or might be missing (due to something that happened).
Let us try to use it. We will need to slightly refactor our class.

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

Since now we want to showcase that the creation of the `FirstName` class can
fail we need to use separate static function instead of constructor.
Constructor cannot return different type that the class it belongs to. And we
want to return `std::optional<FirstName>` to indicate that possible failure.
To do that we make our constructor private and call it from the new static
public function that performs the validation. If the validation succeeds we
return the valid `FirstName`, else we return `std::nullopt` which represents
missing value. Let us take a look on how using this class might look now. First
we replace the call to the constructor, with the call to the `Create` function.

```c++
Customer parse_customer(/* ... */) {
    // …
    auto first_name = FirstName::Create(some_value);
    use_first_name(first_name); // Wrong type!
}
```

This code will not compile yet. Now whenever someone wants to create value of
type `FirstName` they must check whether the creation succeeds. There will be
no unhandled exceptions anymore. The proper way to do this will look as
follows.

```c++
Customer parse_customer(/* ... */) {
    // ...
    auto first_name = FirstName::Create(some_value);
    if (first_name.has_value()) {
        use_first_name(first_name);
    } else {
        // handling code
    };
    // ...
}
```

To summarize, we addressed the issue that users were not aware of the
possibility of failure without looking into implementation. We also moved the
possibility of error from a run-time to compile-time. In other words, we made
our code less error-prone and achieved better documentation with code itself.

A question might arise, what if our validation would involve different checks
and we would like to inform the user what exactly went wrong instead of just
returning a missing value. We could simply use a variant that will store one of
two types, one will represent a possible error, and the second one will
represent a desired value.

```c++
enum class ValidationError {
    NameContainsInvalidCharacters,
    NameTooLong,
};
 
using ValidationResult = std::variant<ValidationError, FirstName>;
```

Now we just need to reflect this in our `Create` method.

```c++
ValidationResult FirstName::Create(std::string name) {
   if (!is_valid_name(name)) {
       return ValidationError::NameContainsInvalidCharacters;
   }
   if (is_too_long(name)) {
       return ValidationError::NameTooLong;
   }
 
   return FirstName(name);
}
```

Many other languages like Rust and Haskell have had such types for a long time
already, with much better language support for them. There is ongoing work to
add a type that will do what we need to the C++ standard library with all
auxiliary functionality attached, unfortunately there is no expected time of
arrival yet. Nonetheless like I presented already, we can build our own type
like that with `std::variant`.

## Summary

I hope that after reading this article you can see the benefits of properly
utilizing a type system. We refactored our initial code in a way that moves
possible errors from run-time to compile-time. We made it more obvious of what
values are expected inside our system and improved its integrity and
maintainability. At last, we now have explicit documentation for our functions,
not only about what values they take but also about the effects they carry. The
techniques I presented are only the basic ones when it comes to improving your
codebase with types. Depending on the language you use, some things might be
easier to do, some might not. If you are interested in more ways of bending the
compiler to your will, I encourage you to read some books on type-driven
development and domain driven design. There are many books that dive much
deeper into this topic and will show you some more sophisticated methods.

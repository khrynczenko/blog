+++
title = "SOLID #3: The Liskov substitution principle"
date = 2021-01-09

+++

## The Liskov substitution principle

The Liskov substitution principle (LSP) is here to help us build good
inheritance hierarchies that support the previous two rules (the single
responsibility and the open/close principle) and work in a way that clients
expect them to.

Before going in I want to mention that I find this principle to be not that
helpful, and not in unison with today's good standard practices. For me this
principle needs to be followed only when coding in a way that I would not,
i.e., nonflat class hierarchies, not taking advantage of the type system.

## Formal definition

Here is the formal definition for the Liskov substitution principle.

> If S is a subtype of T, then objects of type T may be replaced with objects
> of type S, without breaking the program.

Basically, it says that we can replace parent classes with their child classes
and the program will run. I believe that this doesn't necessarily tell the
whole story.
For me, it is not only that the subtype must not break the program. For me
it has to also behave as expected from a subclass of such a type, e.g.:

```python
class Logger(ABC):

    @abc.abstractmethod
    def log(message: Text):
        pass
    
class BadLogger(Logger):

    def log(message: Text):
        sys.exit(0)
        # This satisfies the interface and does not break the program,
        # but it obviously has nothing to do with logging.
```

## LSP rules

LSP provides several rules regarding the types and subtypes in the hierarchies.

### Conract rules

These rules tell us what are the restrictions that are put on subtypes by
the supertype.

- Preconditions cannot be stregthened in a subtype.
- Postconditions cannot be weakened in a subtype.
- Invariants of the supertype must be preserved in a subtype.

#### Preconditions

If a supertype states that for example one of the argumets provided must
be positive, then all of the subtypes should not change that. The subtypes,
must neither strengthen nor weaken this contract.

```python
class RegularShipping:
    def ship(address: Address, kilograms: int):
        if kilograms < 1:
            throw ValueError("Kilograms shipped must be a postive value.")
        # ...

class QuickShipping(RegularShipping):
    def ship(address: Address, kilograms: int):
        if kilograms < 1 and kilograms > 10 :
            throw ValueError("For quick shipments packages must be under 10 kilgorams")
            # This strengthens the contract of the superype
        # ...
```

#### Postconditions

Postconditions are made to check whether object is still in valid state after
the operation. Instead of being at the beginning of the method, they are
at their end.

```python
class RegularShipping:
    def ship(address: Address, kilograms: int):
        if kilograms < 1:
            throw ValueError("Kilograms shipped must be a postive value.")

        # Shipping code
        if not self.has_shipped:
            throw ValueError("Somehow the packaged has not beed shipped.")

class QuickShipping(RegularShipping):
    def ship(address: Address, kilograms: int):
        if kilograms < 1:
            throw ValueError("Kilograms shipped must be a postive value.")
            # This strengthens the contract of the superype

            # Shipping code
            # no validation code
```

#### Data invariants

Data invariants mean that some data within the object must hold to some rules
(for example value of `tries` must not exceed 10), and those rules must
be adhered to by the subtypes.

```python
class HttpRequest:
    def __init__(self):
        self._tries = 0 # This is an invariant, it cannot exceed 10

    def request(url) -> Optional[Response]:
        if self._tries > 10:
            throw RuntimeError("Exceeded 10 tries already.")

        response = send_request(url) 
        if response is None:
            self._tries += 1
            return None
        return response

class HttpsRequest(HttpRequest):

    def request(url) -> Optional[Response]:
        # Invariant may not be holded because there are no chceks for it.

        response = send_request_secure(url) 
        if response is None:
            self._tries += 1
            return None
        return response
```

#### How types can save us?

I believe that the most problems that the LSP tries to solve can be avoided
with simple good use of types and having flat hierarchies (no inheritance, i.e.,
only implementing interfaces).

For example all of the preconditions, post conditions, and data invariants
can be defined in types instead of being checked at runtime all over the place.
This also resutls in self-documenting code.

```rust
struct Kilograms{
    value: usize
};

impl Kilograms {
    pub new(value: usize) -> Option<Kilograms> {
        if value == 0 {
            None
        } else {
            Some(Kilograms{value})
        }
    }

    pub get_value(&self) -> usize {
        self.value
    }
}

trait Shipping {
    fn ship(&self, address: Address, weight: Kilograms);
    // No need for preconditions :D
}

```

Doesn't this look much better? We don't need to remember to check if
value of the Kilograms is valid in the code that uses it. Because it is a type
we can be sure that it is. This pattern of enforcing some rules for
**simple values** (integers, floats, strings, etc.) is also known as a `newtype`
pattern.

### Liskov type system rules

- There must be contravariance of the method arguments in the subtype
- There must be covariance of the return types in the subtype
- No new exceptions are allowed

#### Contravariance

This has to do with the fact that you should use supertype in the method
parameters instead of relying on a subtype.

```python
class RegularShipping:
    def ship(address: Address, kilograms: int):
        if kilograms < 1:
            throw ValueError("Kilograms shipped must be a postive value.")
        # ...

class QuickShipping(RegularShipping):
    def ship(address: Address, kilograms: int):
        if kilograms < 1 and kilograms > 10 :
            throw ValueError("For quick shipments packages must be under 10 kilgorams")
            # This strengthens the contract of the superype
        # ...

def ship(shipping: QuickShipping): # BAD
    # ...
def ship(shipping: RegularShipping) # GOOD
    # ...
```

#### Covariance

This has to do with the fact that you can return subtypes in
a functions that have supertype in their signature.

```python
class RegularShipping:
    def ship(address: Address, kilograms: int):
        if kilograms < 1:
            throw ValueError("Kilograms shipped must be a postive value.")
        # ...

class QuickShipping(RegularShipping):
    def ship(address: Address, kilograms: int):
        if kilograms < 1 and kilograms > 10 :
            throw ValueError("For quick shipments packages must be under 10 kilgorams")
            # This strengthens the contract of the superype
        # ...

def make_shipping(details) -> RegularShipping: 
    # ...
    return QuickShipping()

```

#### No new exceptions

This is rather self-explanatory, do not use new exceptions in your
subtypes. End of story.

For me it is easy becuause I don't use exceptions at all. To me they are more
trouble than it is worth.

## Conclusion

The Liskov substitution principle guides us with a set of rules to
create predictable inheritance hierarchies. When these
rules are followed, the users of the aformentioned hierarchies will not be
suprised with diverging behavior.

PS. To be honest I don't think this is a super useful rule since most of the
problems can be removed completly with correct use of the type system and
flat hierarchies.

In essence:

- instead of holding invariant, preconditions and postconditions with the
  runtime checks, represent these within the type system,
- make all non leaf classes abstract.  

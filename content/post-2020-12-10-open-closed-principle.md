+++
title = "SOLID #2: The open/closed principle"
date = 2020-12-10

+++

## The open/closed principle

The open/close principle, although very simple, can be confusing because of the
name that makes it somewhat confusing. At one point it indicates restrictions,
on the other some freedom. There are two common definitions when it comes
to this principle. One is called the Meyer definition and it states that:

> Software entities should be open for extension but closed for modification.

This is what most people know, and you will hear from when asking about the
open/closed principle. Unfortunately, it does not explain much, and just
remembering this definition, does not help in anything. The other one is called
the Martin definition but I would hardly call it definition,
rather it is an explanation of the meaning behind it. Since I would like to
explain it myself, the way I understand this principle (hey this is my diary)
I will not put it here. Let's digest the principle itself.

### Open for extension

When I think of code that is open for extension, I think of code that is easy
to adapt to new requirements. I think of code that provides enough endpoints
for me to provide new functionality without changing the existing
implementation.
Why is this important? Because I don't want to change the existing
implementation,
why would I? When I am changing already existing fragments of code I need to
put a lot of focus so I don't break anything. It is much easier, and probably
much less error-prone to add new code, instead of changing extending one.
Remember we are talking about adapting to new requirements, not fixing bugs,
so adding code is usually the only way anyway.

### Closed for modification

What about code being closed for modification? This rule makes
sense only when put against the first tone. In this context source code
should be closed for modification, **when it comes to extending the
behavior** only. We are not talking about fixing bugs, in such scenarios, it
is obviously okay to modify existing code. This rule basically says that
we should extend our solution space through extension points only, thus
adding new code.

### How this works

Let's look at how code that doesn't come with extension points
(thus not open for extension) looks like.
In the spirit of the first *SOLID* post, let's look at our
deserialization algorithm, in its first original implementation.

```python
@dataclass
class User:
    name: Text
    email: Text

def deserialize_user(serialized_user: Text) -> Optional[User]:
    # DECRYPTION
    decrypted_letters = map(lambda letter: chr(ord(letter) + 1),
                            serialized_user)
    decrypted_text = "".join(decrypted_letters)

    # PARSING
    parsed_values = decrypted_text.split("$")
    if len(parsed_values) != 2:
        return None
    user_name = parsed_values[0]
    user_email = parsed_values[1]
    return User(user_name, user_email)
```

This code is not in any way open for extension. Imagine that there is a new
requirement that the decryption algorithm, is different for clients that use
a commercial license of your software. With such code, there is not another
way around it, and you need to make a lot of changes to the existing code.

Now let's look at code that does provide extension points, through interfaces.
```python

@dataclass
class User:
    name: Text
    email: Text


class Decryption(ABC):
    @abc.abstractmethod
    def decrypt_user(self, serialized_user: Text) -> Text:
        pass


class PlusOneAsciiDecryption(Decryption):
    def decrypt_user(self, serialized_user: Text) -> Text:
        decrypted_letters = map(lambda letter: chr(ord(letter) + 1),
                                serialized_user)
        return "".join(decrypted_letters)


class Parsing(ABC):
    @abc.abstractmethod
    def parse_user(self, decrypted_user: Text) -> Optional[User]:
        pass


class DollarSplitParsing(ABC):
    def parse_user(self, decrypted_user: Text) -> Optional[User]:
        parsed_values = decrypted_user.split("$")
        if len(parsed_values) != 2:
            return None
        user_name = parsed_values[0]
        user_email = parsed_values[1]
        return User(user_name, user_email)

def deserialize_user(decryption: Decryption,
                     parsing: Parsing,
                     serialized_user: Text) -> Optional[User]:
    return parsing.parse_user(decryption.decrypt_user(serialized_user))

if __name__ == "__main__":
    decryption = LoggedDecryption(PlusOneAsciiDecryption())
    parsing = LoggedParsing(DollarSplitParsing())
    print(deserialize_user(decryption, parsing, "trdq#dl`hk?cnl`hm-bnl"))
```

With that we can add a new decryption algorithm without any hassle, we don't
even need to take a look at the implementation of the other decryption
algorithm. **We can extend the behavior of our code,
without modifying the existing one!**

### Extension points

So in essence we want to provide extension points in our codebase that would
provide the means to extend it, thus making it more adaptable in face of changing
requirements. There are many ways of doing that, most notably:

- virtual methods
- abstract methods
- interfaces

I don't see a point in explaining all of them, because I think the
`interfaces`, are the way to go. They provide the most flexibility, are more
intuitive, and less error-prone.

### Protected (predicted) variation

That this all means that we need to use interfaces everywhere? Of course not.
Interfaces are great but they also come with costs. They introduce some level
of indirection which always comes with some overhead. How to choose then, if
to provide (or not) extension points, for new code.

In essence, you need to weigh how possible it is that the code under
consideration is likely to change in the future. It is not an easy task.
Figuring that out often requires input from the stakeholders, domain experts,
and other parties involved. It comes easier with experience. In my opinion,
as a rule of thumb, if you are not sure, lean towards thinking that the code
is likely to change in the future.


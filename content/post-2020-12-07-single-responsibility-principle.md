+++
title = "SOLID #1: The single responsibility principle"
date = 2020-12-07

+++

## SOLID

For a long time now, I have been following just a couple of rules, or rather
intuitions, when I am coding. Keep code simple, don't repeat yourself, don't
implement what you don't need (KISS, DRY, YAGNI). I was focusing on making my
code easy to understand, and on making code adaptable
by writing to interfaces, and using dependency injection. I still believe that
simply following these few rules can get you very far. They alone, in my
opinion, make a very competent programmer (as long as you are conciously,
adhering to them).

Unfortunately, sometimes the actual means of conforming to these principles can
be unclear. Not only that, different situations require a different approach.
Because of that, even though the principles themself are simple, they are not
as easy to conform to as it seems. It seems that some guidance, would be really
helpful in times of need.

That is why I have decided to revisit the SOLID principles, which in my opinion
focus mostly on making your code adaptable, and simple at the same time. In the
following blog posts, I will revisit each principle and try to put emphasis on
why and how it can be used to our advantage.

Let's start with the single responsibility!

## The single responsibility principle

I think there are two groups of people when it comes to describing what
actually is the SRP. One group would say that it is about your
units of code doing only one thing. The other group would say that SRP
states that code should have only one reason to change. I fall in the latter,
but I will try to apply both ways of thinking about the SRP.

You would ask then, "but what is a unit of code, and what exactly is one thing?".
That is why I like to follow the second approach because this is not often
clear. A unit of code might be either a module, class, or function.
As to the one thing, for me, it means that unit of code should do only that
what its name says. For example, if the function name is `deserialize_user`,
it shouldn't be doing decryption, and parsing since this is not its
responsibility, even though it is needed in a deserialization process.

For the sake of example let's say that we actually want
to deserialize user objects. This process involves decrypting and then parsing
the serialized string.
Decryption involves taking a string and changing each letter with the next
character in the ASCII table. Parsing involves splitting a decrypted
string into two parts user name, and user email, where a dollar mark is a
separator between them.

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

```python
print(deserialize_user("trdq#dl`hk?cnl`hm-bnl"))
>>> User(name='user', email='email@domain.com')
```

I think we can see clearly that this function does two things,
it does decryption, and it does parsing. We can see that this function
has also more than one reason to change, actually three:

- it needs to change when decryption algorithm changes
- it needs to change when parsing algorithm changes
- it needs to change when deserialization algorithm changes, e.g., additional
  initial step of decoding has to be performed

Let's make the simplest change that we can think of, i.e., extract
functions.

```python
def decrypt_user(serialized_user: Text) -> Text:
    decrypted_letters = map(lambda letter: chr(ord(letter) + 1),
                            serialized_user)
    return "".join(decrypted_letters)


def parse_user(decrypted_user: Text) -> Optional[User]:
    parsed_values = decrypted_user.split("$")
    if len(parsed_values) != 2:
        return None
    user_name = parsed_values[0]
    user_email = parsed_values[1]
    return User(user_name, user_email)


def deserialize_user(serialized_user: Text) -> Optional[User]:
    return parse_user(decrypt_user(serialized_user))
```

You might be wondering how much has actually changed. At first glance, not a
lot. It almost looks like we just moved parts of the code to other places.
That is not the
only effect this change has produced. Now even though it might not initially
look
like it, the `deserialize_user` function does only one thing. It does not
perform any decryption or parsing. Instead, it delegates these to the functions
**responsible** for that. So how many reasons to change exist for this function
now? Only one:

- it needs to change when deserialization algorithm changes, e.g., additional
  initial step of decoding has to be performed.

The other two, namely:

- it needs to change when parsing algorithm changes,
- it needs to change when deserialization algorithm changes, e.g., additional
  initial step of decoding has to be performed,

are now reasons for their new corresponding functions. To summarize each of
our functions has only **one** reason to change now.

Is that all the story? Not really. Our refactoring definitely brought clarity,
and it would be much easier to reason about this code, and apply necessary
modifications, like bug fixes, but there is still room for improvement.
Imagine that we would like to log when decryption and parsing,
starts and finishes. Lets start with the obvious and slap logging into the
deserialazing function.

```python
def deserialize_user(serialized_user: Text) -> Optional[User]:
    logging.log(INFO, "Decryption process started...")
    decrypted = decrypt_user(serialized_user)
    logging.log(INFO, "Decryption process finished...")
    logging.log(INFO, "Parsing process started...")
    parsed = parse_user(decrypted)
    logging.log(INFO, "Parsing process finished...")
    return parsed
```

Again this looks okay at first, but we added one more responsibility to the
`deserialize_user` function. It has one more reason to change now, i.e.,
when logging process needs to change. Now extracting a functions doesn't look
like a feasible solution. There is another one.

We can make our code more adaptable, it is, resiliant in face of
changing requirements (like requirement for logging).
Since we can see that our function now delegates
responsibilities for decryption and parsing (and soon logging),
we could have them injected into the `deserialize_user` function as a
dependency. But before doing that let's start using interfaces
(or traits or abstract classes, or whatever your language
provides), because I think for most people they are easier to
understand than just using functions. Forget about logging for now, we will
come back to it soon enough.

```python
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

```

We extracted interfaces for our decryption and parsing algorithms and made them
dependencies of our `deserialize_user` function. This gives some crucial
benefits like:

- ability to exchange these algorithms without modifying existing implementations
- decide on what algorithm to use at runtime,
- ability to extend the functionality of existing classes with decorators.

The third benefit is crucial for our logging functionality. Decorators are
really handy when we want to extend functionality without introducing more
responsibilites to some code. It is a common design pattern, an there are
many explanations on how it works that you can find on the internet.
We can now have another class (unit of code) be responsible for logging only.
Let's see how this would play out in the code.

```python
class LoggedDecryption(Decryption):
    def __init__(self, decryption: Decryption):
        self._decryption = decryption

    def decrypt_user(self, serialized_user: Text) -> Text:
        logging.log(INFO, "Decryption has started...")
        decrypted_user = self._decryption.decrypt_user(serialized_user)
        logging.log(INFO, "Decryption has finished...")
        return decrypted_user


class LoggedParsing(Parsing):
    def __init__(self, parsing: Parsing):
        self._parsing = parsing

    def parse_user(self, decrypted_user: Text) -> Optional[User]:
        logging.log(INFO, "Parsing has started...")
        parsed_user = self._parsing.parse_user(decrypted_user)
        logging.log(INFO, "Parsing has finished...")
        return parsed_user
```

As you can see `LoggedDecryption`, and `LoggedParsing` take responsibility
for logging, and logging only. Lets look at it all at once.

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


class LoggedDecryption(Decryption):
    def __init__(self, decryption: Decryption):
        self._decryption = decryption

    def decrypt_user(self, serialized_user: Text) -> Text:
        logging.log(INFO, "Decryption has started...")
        decrypted_user = self._decryption.decrypt_user(serialized_user)
        logging.log(INFO, "Decryption has finished...")
        return decrypted_user


class LoggedParsing(Parsing):
    def __init__(self, parsing: Parsing):
        self._parsing = parsing

    def parse_user(self, decrypted_user: Text) -> Optional[User]:
        logging.log(INFO, "Parsing has started...")
        parsed_user = self._parsing.parse_user(decrypted_user)
        logging.log(INFO, "Parsing has finished...")
        return parsed_user


def deserialize_user(decryption: Decryption,
                     parsing: Parsing,
                     serialized_user: Text) -> Optional[User]:
    return parsing.parse_user(decryption.decrypt_user(serialized_user))

if __name__ == "__main__":
    decryption = LoggedDecryption(PlusOneAsciiDecryption())
    parsing = LoggedParsing(DollarSplitParsing())
    print(deserialize_user(decryption, parsing, "trdq#dl`hk?cnl`hm-bnl"))
```

As you can see, now each of our units of code (classes and functions) has
only one single responsibility. Although the number of lines of code has
increased, we got much more in return. Each part of the code is now much more
digestible. We can go to parsing and see how it works without even thinking on
how decryption, or loggin works and vice versa. We can extend our functionality,
without introducing new responsibilities by means of decorators. You can imagine
that besides logging, we could chain many different decorators. We could not
only log, but also measure performance, persist data in a database, or send it
over the network. And each of these would be completley seperated, and could be
used in different combinations with different implementations. And be decided on
runtime. Is not that great?

To summarize, the single responsibility principle states that each unit of code
like function or class should have only one reason to change. In other words,
each unit of code should do one thing and one thing only. The simplest way of
achieving that is by extracting functions (or classes). This often improves
clarity, but in order to improve adaptability, i.e., resilience in face of
changing requirements, we should adapt our code to use interfaces. Interfaces
give us many benefits, such as loose coupling, runtime dependency injection,
and improved extendibility with decorators.

I hope this shows how applying single responsibility principle can benefit
our codebase. This was rather small example, but in real world scenarios
this approach really shines.

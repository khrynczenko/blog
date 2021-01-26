+++
title = "SOLID #5: The dependency inversion principle"
date = 2021-01-13

+++

## The dependency inversion principle

The dependency inversion principle states that:

- high-level modules should not depend on low-level modules, both should
depend on abstractions
- abstractions should not depend on details, details should depend on
abstractions.

In essence, it is all about keeping our code modular, properly separating
interfaces from implementations is vital to maintain modularity.

### The entourage anti-pattern

There is an anti-pattern which is known as the **Entourage anti-pattern**
which is a problem where supposedly importing one code element to the
namespace brings also the neighboring elements (the entourage).

Let's look at some example.

```python
# parser.py

import json

class JSONParser(ABC):

    @abc.abstractmethod
    def parse(json: str) -> dict:
        pass

class DefaultParser(JSONParser):
    def parse(json: str) -> dict:
        json.loads(json)

```

```python
# user.py

import parser

class User:
    def __init__(json_parser: JSONParser):
        # ...

    def read(json: str):
        # ...

```

If you look carefully you will notice that in the user module
we implicitly now made another dependency, the dependency on the `json` package.
This is not desirable since what we really use is just the interface and the
interface alone does not need the `json` package to be present.

I know, I know, if the `parser` module lies in the same package like ours, that
is not much of a problem but if the interface lies in a separate package
then when we add it we will be forced to download the `json` dependency anyway.

As a general rule, implementations should be put into separate crates (rust),
assemblies (C#), packages (python).

### The stairway pattern

The solution to the aforementioned problem is to put interfaces and their
corresponding implementations in different packages. Something like in
the example below.

```python
# parser.py

class JSONParser(ABC):

    @abc.abstractmethod
    def parse(json: str) -> dict:
        pass

```

```python
# parser_impl.py

import json

from parser import JSONParser

class DefaultParser(JSONParser):
    def parse(json: str) -> dict:
        json.loads(json)

```

```python
# user.py

from parser import JSONParser

class User:
    def __init__(json_parser: JSONParser):
        # ...

    def read(json: str):
        # ...

```

Now the `user` module does not implicitly depend on the `json` package.
If you would try to represent this way of separation of interfaces and
implementations using a UML you would see something similar to a
stairway, hence the name. This way of organizing your
interfaces/implementations make it way easier to incorporate your code
for your clients.

There is much more to the dependency inversion but I will stop right
here because I think that this is the essence.

## Special notes

I know that I rushed over this and the last principle. I am really time
constrained these days and writing posts like the one I did on the single
responsibility principle takes me a couple of hours at least.

If you would like to know more I highly recommend the "Adaptive Code" by
Gary Mclean Hall. All my understanding and some of the examples I presented
in this blog series are based on this book.
book. In my opinion it is *THE* book on the **SOLID** principles.

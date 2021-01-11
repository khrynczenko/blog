+++
title = "SOLID #4: The Interface segregation principle"
date = 2021-01-11

+++

## The Interface segregation principle

I admit that the name of this principle kind of confused me the first time
I read it. Fortunately it can be summarized in a one sentence. Keep your
interfaces small.

## Small interfaces

Why smaller interfaces are better than bigger ones? Beside obvious stuff
like that the smaller interfaces are easier to follow, easier to develop
for, and induce less cognitive overhead, the one big advantage of smaller
interfaces is that they provide better support for decoration and thus the
single responsibility principle.

Let's say that we have interface that allows our clients to manage some
persistence storage.

```rust
trait Storage<T> {
    fn add(&mut self, value: T);
    fn delete(&mut self, value: T);
    fn read(&self, id: usize) -> T;
    fn read_all(&self) -> &[T];
}
```

Now imagine we want to ask the user if he really wants to delete a value from
the storage before outright deleting it.

```rust
pub struct AskToDeleteStorage<T> {
    storage: Box<dyn Storage<T>>,
}

impl<T> AskToDeleteStorage<T> {
    pub fn new(storage: Box<dyn Storage<T>>) -> AskToDeleteStorage<T> {
        AskToDeleteStorage { storage }
    }
}

impl<T> Storage<T> for AskToDeleteStorage<T> {
    fn add(&mut self, value: T) {
        self.storage.add(value);
    }
    fn delete(&mut self, value: T) {
        let mut user_input = String::new();
        std::io::stdin().read_line(&mut user_input).unwrap();
        if user_input == "yes" {
            self.storage.delete(value);
        }
    }
    fn read(&self, id: usize) -> T {
        self.storage.read(id)
    }
    fn read_all(&self) -> &[T] {
        self.storage.read_all()
    }
}
```

You can see that although we achieved what we wanted, we had to implement
each method even though we were changing behavior of only one of them. I guess
we could say that it is not that bad, but if there would be a couple more
decorators, or/and couple more methods, this would be a really noneffective
way of spending your time. Not to mentiond that if you write tests for these
decorators now you have to write a couple more which don't actually check
that much.

Here comes the interface segregation. We want to divide this big interface
into a couple of smaller ones so that this problem goes away.

```rust
trait Add<T> {
    fn add(&mut self, value: T);
}

trait Delete<T> {
    fn delete(&mut self, value: T);
}

trait Read<T> {
    fn read(&self, id: usize) -> T;
    fn read_all(&self) -> &[T];
}
```

Now we can happily decorate just the method we wanted without touching
the other ones. This is really handy. Image you would want to ask before
deleting, add caching to reading, or aduit on adding. Now you can have
seperate decorator that does just that and there is no need to write so much
boilerplate anymore.

```rust
pub struct AskToDeleteStorage<T> {
    storage: Box<dyn Delete<T>>,
}

impl<T> AskToDeleteStorage<T> {
    pub fn new(storage: Box<dyn Delete<T>>) -> AskToDeleteStorage<T> {
        AskToDeleteStorage { storage }
    }
}

impl<T> Delete<T> for AskToDeleteStorage<T> {
    fn delete(&mut self, value: T) {
        let mut user_input = String::new();
        std::io::stdin().read_line(&mut user_input).unwrap();
        if user_input == "yes" {
            self.storage.delete(value);
        }
    }
}
```

It is okay now to unify these interfaces into a single class (but please don't
unify them in a single interface, that  would be reverting all the work we
have done).

```rust
pub struct Storage<T> {
    storage: Box<dyn Delete<T>>,
}

impl<T> Add<T> + Delete<T> + Read<T> for Storage<T> {
    fn add(&mut self, value: T) {
        todo!()
    }
    fn delete(&mut self, value: T) {
        todo!()
    }
    fn read(&self, id: usize) -> T {
        todo!()
    }
    fn read_all(&self) -> &[T] {
        todo!()
    }
}
```

That's it. I hope you enjoyed this quick dive into interface segregation. Let's
keep our interfaces segragated in order to keep them adaptable.


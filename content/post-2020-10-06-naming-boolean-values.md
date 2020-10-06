+++
title = "Naming boolean variables"
date = 2020-10-06

+++

## The problem

I must admit that even though I have been programming for some time already,
I still have a problem with naming things. I think I am getting better
at it, but sometimes I mess up. I found myself having trouble with that,
especially when it comes to boolean variables. Nonetheless, I think lately
I have somewhat figured it out, and I would like to share my silly epiphany
here. I will start with an example that shows what bad names for booleans look
like (in my opinion) and then move to what I consider a better approach.

## Misleading boolean names

So when we consider names for the boolean values to be poorly chosen? As with
any other names it happens when it is unclear what `true` or `false` really
mean. So as with any other names the problem occurs when name can
be wrongly understood.

Lets start with a simple class responsible for monitoring status of
another running process on seperate thread.

```rust
struct ProcessMonitor {
    # ...
    monitoring: bool,
}

```

So what does the `monitoring` variable repesents? Is it that the monitoring
has started already if it is `true`? Or maybe it should be started if
it is true. The truth is we cannot know without diving deeper into
implementation.

```rust
struct ProcessMonitor {
    # ...
    monitoring_started: bool,
}

```

Now it is clear that when `monitoring_started` is set to `true` the
monitoring is already being performed.

```rust
pub struct Settings {
    # ... more settings
    long_breaks: bool,
    auto_next_period: bool,
}
```

For the second example, the `Settings` structure is responsible for keeping
user-defined options
for the Pomodoro application. There are three types of *periods* in pomodoro
application, i.e., *work*, *short break*, and *long break*.
The user should be able to set whether he wants
to have long breaks between work and short break intervals. Also, they should
be able to set that if when one period ends the timer for the next one should
start immediately without manual intervention.

Here we have two variables and both of them can be easily misunderstood
(especially without having the context I have provided). `long_breaks`
might be seen as if all the *breaks* are longer than usual, or maybe that
all short breaks are converted to the longer ones. `auto_next_period` could
mean that the user does not have to choose the next period by himself. Anyhow
any of those assumptions would be wrong, and that is because the names are
poorly chosen.

```rust
pub struct Settings {
    # ... more settings
    are_long_breaks_included: bool,
    does_next_period_start_automatically: bool,
}

```

Now these names look much better and are much clearer about what they
represent. There is one problem. They no longer start with nouns, instead
they use verbs wich is the common way to define functions.

```rust
pub struct Settings {
    # ... more settings
    are_long_breaks_included: bool,
    does_next_period_start_automatically: bool,
}

impl Settings {
    pub fn are_long_breaks_included(&self) -> bool {
        return are_long_breaks_active; # ERROR!
    }
    pub fn does_next_period_start_automatically(&self) -> bool {
        return does_next_period_start_automatically; # ERROR!
    }
}

```

This can be fixed by changing the sentence (oh yes we made variable name
into a sentence) from being interrogative to declarative.

```rust
pub struct Settings {
    # ... more settings
    long_breaks_are_included: bool,
    next_period_starts_automatically: bool,
}

impl Settings {
    pub fn are_long_breaks_included(&self) -> bool {
        return long_breaks_are_included;
    }
    pub fn does_next_period_start_automatically(&self) -> bool {
        return next_period_starts_automatically;
    }
}

```

Finally, we have obtained something that is clear, much harder to
misunderstood and conforms with common approach to naming things.
In summary it is as if with naming all the entities that are represented
in code. We should strive for being as clear as we can without introducing
too much noise. It is still hard for me too grasp why I had such problem
with booleans but I believe I have somewhat tackled that problem and
have better understating of it.

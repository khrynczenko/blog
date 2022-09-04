+++
title = "Chapter 1: Introducing Domain-Driven Design"
date = 2022-09-04

+++

## Introduction

I want to understand DDD better because from what I have seen so far it
is very well suited approach for business focused developments. I already
read once the book that I will follow in this series of posts which is called
"Domain Modeling Made Functional" by Scott Wlaschin. But reading is not the
same as deliberate practice. Also the subject is complex enough that it
warrants re-reading on its own. This time I will be more insistent on
formulating my knowledge into words and even more importantly I will
use that knowledge to implement an actual application.

This application will be a simple one. It will be a digital daily notebook.
Nothing more, nothing less. A user can write a note for each day in the
calendar. A user can login, write a note for that day, a day in the future, a
day in a past, read a note from any day, edit a note from any day. Why I would
like something like this? Personally when I work I always note what I did
on the day and I write on what I want to do the next day. It is like a little
history for me I can go back to as well. I guess this is the MVP.

## Introducing Domain-Drive Design

In the introduction to the chapter, I like how the author compares software
development to a pipeline with an input which is **requirements** and an
output which is the **final delivered product**. When we think of software
development in that way the rule **garbage in, garbage out** applies. If
the requirements (input) are unclear then how can we expect the output to
be correct?

DDD is supposed to minimize the *garbage in* part by emphasizing on
**clear communication** and **shared domain knowledge**.

All that said DDD is not a tool that can be applied to every problem.
Different types of software (games, systems software, etc.) might be better
suited for other approaches. However, the business and enterprise software
where non-technical teams must be included in the collaboration usually works
incredibly well with DDD.

## The importance of shared model

It is of utmost importanec for developers to understand the problem at hand.
Without that understanding, they won't be able to produce a suting solution.

Many existing software development processes try to bridge the gap of
understanding by writing formal specifications that capture all the details.
Unfortunately, this creates a distance between the people who understand the
problem (domain experts) and people who implement the solution (developers,
ui/ux experts, testers, etc.).

That lacking process could look like this. *Domain experts* communicate
with *business analysts* to create *requirements document*. That document
is forwarded to an *architect* who then creates a *design document*. This
is then forwarded to the *development team* who does the actual implementation.

We can see that in this approach the distance between the *domain experts*
and the *development team* is maximized. Because of this a lot of domain
knowledge is lost in translation. The author says that the initial message is
distorted. A mismatch between the requirements and the solution becomes real.

A much better one is to put *developers* and *domain experts* next to each other.
This is more aligned with todays *agile* development. But this still entails
that the developers perform a translation of the domain experts' model
into code. The code loses the subtleties of the domain model and with
each passing day, diverges from that model. Code starts to less resemble
the concepts of the domain and misunderstanding will pop up.

Finally, what is recommended is to put *domain experts*, *development team*,
*stakeholders* and most importantly the code together. The code shall
represent a shared mental model. No translation, the code itself ought to
represent the domain exactly as it is put out.

What are the benfits of aligning the software model with the business domain?

- *Faster time to market*. Shared model makes it easier to develop correct
  solution quicker.
- *More business value*. A solution that is aligned with the problem means
  happier customers.
  > Krzysztof: This is rather vague?
- *Less wate*. Clear requirements mean less time wasted in misunderstandings
  and reworks.
- *Easier to maintance and evolution*. Making changes to the code is easier and
  less error-prone. New team members are abler to come up to speed faster.

So how do we create a shared model? Here are the guidelines.

- **Focus on business events and workflows rather than data structures.**
- **Partition the problem into smaller subdomains.**
- **Create a model of each subdomain in the solution.**
- **Develop a common language (Ubiquitous Language) that is shared between
everyone involved in the project and is usedf everywhere in the code..**

## Understanding the Domain Thtough Business Events

We should focus not on the data we operate on but on the transformations
that occur inside the domain. The values of the business is hidden inside
those transformations. What *triggers* those transformations are **Domain
Events**. This is an important part that we need to capture in our design.
Those events are most often starting points for all business processes.
For example, an `OrderFormReceived` event might kick off an order-taking
process that processes the form.

### Using Event Storming to Disover the Domain

To discover the events inside our domain we do the **Event Storming**. We
gather all the domain experts, developers, stakeholders, and pretty much all
the people involved. Then people write out their event and what workflows
they trigger.

This approach allows us to:

- build a shared model of the business,
- build awarness of all the teams that exist,
- finding the gaps in the requirements,
- make connections that exist between teams,
- awarness of reporting requirements (kind of ui requirements).

There are many different terms to desribe business activites, i.e.,
**workflows**, **scenarios**, **use cases**, **processes**. They are often
used interchangibly but the devil is in the details.

- A **scenario** describes a goal that a customer/user wants to achieve,
such as placing an order. It corresponds to a **story** in agile development.
What is important is that they are **user centric**.
- A **business process** describes the goal of the business instead of focusing
on the user. It has a business centric focus.
- A **workflow** is a detailed descriptions of part of a business. It is a list
of exact steps that an **employee (or software component)** needs to perform
to acomplish a business goal.

### Documenting commands

Once we have defined the events, the next step is to ask what caused them.
Those we call **commands**. **Command** initiates a **workflow**. If that
**workflow** succeeds it emits and **event**.

Command: Make X happen, Workflow: Make X happend, Event: X happened.

![Commands, Events, Worfkflows](/event-command-workflow.png)

> Krzysztof: It is a little bit weir that commands and workflows usually
> have the same name and can be confusing, but they are not exactly the same.
> More on that later.

**Not all events must be associated with a command. Some events might be
triggered by some external system like a scheduler or a monitoring system.**

## Partitioning Domain into Subdomains

A domain can usually be separated into several subdomains. But what is a domain
anyway. The author defines it as:

> A domain is an area in which a domain expert is an expert.

Whatever it means. I like to think that a domain is an area in which an
internal shared model exists that does not crossover to other domains (even
if the language is similar or even the same for some concepts).

## Creating a Solution Using Bounded Contexts

Our solution should only capture the information necessary to solving a
particular
problem, not the whole domain. Thus we have to create a distinction between
**problem space** and **solution space**. **Solution space** extracts only
relevant information from the **problem space** that are necessary for the
implementation.

These solution spaces we call **bounded contexts**.

![Bounded Contexts](/bounded-contexts.png)

### Bounded context

A bounded context is a subsystem in the solution space with clear boundaries
that distinguish it from other subsystems. In other words, it is also an
implementation of a domain.

Bounded contexts usually have a one-to-one correspondence with domains in the
problem space, but it is not a rule. Multiple domains may be
represented by a one boundary context. For example when you use a prebuilt
component that solves problems related to a couple of domains.

It is important that each bounded context has a clear responsibility, because
it usually corresponds to some software component, whether it is a service,
DLL, or something else.

When defining bounded contexts listed by the domain experts, pay attention
to existing team and department boundaries, and don't forget that it is bounded.

### Getting the contexts right

Advices for formulating bounded contexts are:

- **listen to domain experts**,
- **pay attention to existing team and department boundaries**,
- **don't forget the "bounded" part**,
- **design for autonomy**,
- **design for friction-free business workflows**.

## Creating context maps

Context maps define what are the relationships between our bounded contexts.

![Bounded Contexts](/context-maps.png)

They can be more detailed with well defined contracts between the contexts.
More on that in later chapter.

## Creating Ubiquitous Language

Inside the domain we should not have terms that are not domain specific. Things
like `OrderManager` shall not exist. If there is a term that a domain expert
would not understand it should not be there, or at least they should not
be exposed.

It is also **important** to understand that different bounded contexts might
have their own dialects of the Ubiquotious Lanaguage or the same word can
have two different meanings inside them.

It is important to note these differences.

## Summary

- A domain is an area of knowledge that which a "domain expert" is expert in.
- A **Domain Model** is a set of simplifications that represent those aspects
of a domain that are relevent to a particular problem. The domain model is
part of the **solution space**, while the domain it represents is a part of
**problem space**.
- The **Ubiquotious Language** is a set of concepts and vocabulary that is
associated with the domain and shared between everyone involved **and** the
code.
- A **Bounded Cotext** is a subsystem in a solution space with clear boundaries
that distinguish it from other subsystems. It corresponds to a subdomain
in a problem space. A bounded cotext has its own set of concepts and vocabulary
that might be a dialect of Ubiquotious.
- A **Cotext Map** is a diagram that defines relationships between bounded
contexts.
- A **Domain Event** is a record of something that happened in a system. It
often acts as a trigger to a command.
- A **Command** is a request for some process to happen and is triggered by
an event. If the command succeeds the state of the system changes and a new
event is emited.

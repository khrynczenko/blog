+++
title = "Domain Modeling Made Functional #3: A Functional Architecture"
date = 2022-09-13

+++

Here we lay out how to translate the domain we laid out into actual software.

We borrown the terminology from Simon Brown's C4 approach. In this approach
software architecture consists of four parts:

- the **system context** is a a top level, representing the entire system and
consists of number of **containers**,
- the **container** is a deployable unit such as a website, a web service, a
database, etc.,
- many **components** can make up a container (not sure what it is exactly),
- many **modules**/**classes** make up a component.

## Bounded Contexts as Autonomous Software Components

If we have a monolith system, i.e., everything is in a single container
and deployable just on its own, a bounded context could be a simple module
with a well defined public interface. On the other hand each bounded context
could be a separate *C4* container making it a service oriented architecture.
We could go even deeper and present each workflow as a separate *C4* container
and makie it a microservice oriented achitecture. It is all up to us. What is
important is boundaries.

## Communicating Betweem Bounded Contextsare

**Bounded contexts** communicate with each other through events only. The
implementation that handles the communiqation could be a messaging queue or
something else. The handler  that translated events to commands could live on
the boundary of the context.

It could also be done by a separate
[*message router*](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRouter.html)
or [*process manager*](https://www.slideshare.net/BerndRuecker/long-running-processes-in-ddd)

## Transferring Data Between Bounded Contexts

The data (domain objects) that we transfer inside events between bounded
context might look identical to events but in reality it is not the same. This
data is specifaclly designed to be serialized and shared as a part on
interconnected infrastructure. These object are called  **Data Transfer
Objects** or **DTO**s in short. In short inside a `UserRegisteredDTO` DTO
there will be almost the same data as in `UserRegistered` event but it will
be structured differently to suit its purpose.

![Domain Transfer Objects IN](/DTO.png)
![Domain Transfer Objects IN](/DTO-OUT.png)

## Trust Boundaries and Validation

The perimeter of a bounded context acts as a "trust boundary". Anything inside
a context must be valid. Anything outside of it must ba validated.

![Domain Transfer Objects IN](/ddd-gates.png)

At the input gate we always validate the input, things like non-null, etc.
If such validation fails then the workflow is bypassed and error is generated.
The job of the output gate is to ensure that private information is not leaked.

## Contracts Between Bounded Contexts

We strive to reduce copling between contexts as much as possible but we cannot
remove it completely as they in the end work together. This coupling is
decided by a contract and who defines the contract depends.

- **Shared Kernel** - a relationship in which two contexts share some common domain,
so the teams must collaborate.
- **Customer/Supplier** (Consumer Driven Contract) - downstream context defines the
contract, that the upstream context must adhere to.
- **Conformist** - Upstream defines the contract for the downstream.

### Anti-Corruption Layers

Sometimes when we communicate with external systems the do not fit at all
into our domain. For these we create *ACL*s that translate data into valid
objects in our domains. This happens at the *gates*.

## Workflows Within a Bounded Context

Workflow in the and is a single function that takes command object
as input and the output is a list of object events.

![Domain Transfer Objects IN](/ddd-workflows.png)

A worfklow is always contained within a single bounded context.

> If a contract between two bounded context is *consumer driven* then rather
than sending a generic event we would probably have a more specific event
defined fo the consuming context.

**Workflow does not publish any events, it simply returns them!**

### Avoid Domain Events Within a Bounded Context

We should not be listening for event coming from the same bounded context.
It is better to instead just append what handler would to at the end of the
workflow. This mitigates hidden dependencies that can come up.

## Code Structure Within a Bouded Context

Use **onion archtiecture**! The domain should be at the center, and each
outside layer can only depend on the inner layers. The domain should be at the
center, and each outside layer can only depend on the inner layers.

For example *domain layer* being at the center, can be a dependency of a
*service* layer, which the can be a dependency of API, infrastructure and
database layers. To do that use **dependency injection**.

## Keep I/O at the edges

In functional approach we want to work with pure functions and keep the impire
ones at the edges of the architecture. All functions should have
explicit dependencies instead of hidden dependencies (like accessing
a database inside a workflow should no come from some global function but
rather be injected through a parameter). For example read from database
at the beginning of a workflow, not inside it.

[Previous chapter - Understanding the Domain](/posts/post-2022-09-12-domain-modeling-made-functional-2)

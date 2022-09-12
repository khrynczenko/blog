+++
title = "Chapter 2: Understanding the Domain"
date = 2022-09-04

+++


## Interview with a Domain Expert

Once we have finished the event-storming and we have many commands and events
prepared and attached to different domains we can dive deeper into specific
business workflows. For example if we have a command "Place order" and an
event "Order placed" we can dive into the "order placing" worfklow.

In such interview we exchange question and answers with a domain expert. You
could start with:

> Developer: Let's talk about order placing worfklow. What information you need
> to start this process?
> Expert: Well we start with this piece of paper, the order form that customers fill
> out and send us in the mail. We would like customers to fill this form live.

Some advices on doing interviews like these:

- stay at high-level at first and focus on inputs/outputs of the workflow,
- resist the urge to jump to conclusions, **listen**.

### Understanding the Non-functional Requirements

Step back and discuss context and scale of the worklow:

- ask about numbers, how many times the process is performed,
- rensponse times,
- amount of data, etc.

### Understanding the Rest of the Workflow

Dive into details.

### Fighting the Impulse to Do Database-Driven Design

Avoid jumping into designing tables and architecting your databases. You should
let *domain drive* the design, not a database schema. It is better to work
from the domain and to model it without respect to any storage implementation.
If you do this the other way around you will find yourself in situatuation
where you adjust your solution space to accomodate the storage solution and
it should be the other way.

This in DDD tereminology is called **persistence ingorance**. Be persistence
ignorant!

### Fighting the Impulse to Do Class-Driven Design

You may also introduce bias  into your design if you think in terms
of objects rather than the domain. OOP programmers will immediately introduce
interfaces, etc., but domain experts will not understand what they are.

## Documenting the Domain

A simple text-based language can be used for documenting the domain.
For workflows we document inputs, outputs, and business logic.
For Data structures we use records and sum types using *AND* and *OR*. Below
is a simple example.

```text
Bounded context: Order-taking
Workflow: "Place order"
    triggered by:
        "Order form received" event (when quote is not checked)
    primary input:
        An order form
    other input:
        Product catalog
    output events:
        "Order Placed" event
    side-effects:
        An acknowledgement is sent to the customer,
        alongwith the placed order.
data Order =
    CustomerInfo
    AND ShippingAddress
    AND BillingAddress
    AND list of OrderLines
    AND AmountToBill

data OrderLine =
    Product
    AND Quantity
    AND Price

data CustomerInfo = ??? don't know yet
data BillingAddress = ??? don't know yet
```

The advantage of such documentation is that it is easy to read for
non-programmers as well.

## Diving deeper


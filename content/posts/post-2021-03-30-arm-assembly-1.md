+++
title = "32-bit ARM Assembly Basics: Part I - Introduction"
date = 2021-03-30

+++

## Introduction

From time to time I get into a situation where I have two pieces of code
that do the same thing. Usually, I would judge which one is better by
which one is easier to understand, which one is idiomatic, followed
by which one is more adaptable and maintainable. The order is not necessarily
always like that but usually, it is. But what if I cannot decide based on
the aforementioned factors? What is left is performance. In such situations,
I like to dive into the assembly. Of course to reason about the performance of
the code, you do not have to read the assembly produced by the compiler.
Most of the time reasoning on the language level will suffice. Yet there are
situations where the assembly output is the only thing we have left to rely on.
And if you are working on high-performance requiring problems you probably
already know that hot-paths need to be thoroughly analyzed on the assembly level
anyway.

## 32-bit ARM Assembly Basics

Many people think that ARM processor architecture is much newer when compared
to the X86 family of processors. Okay, maybe it was just me. The fact is that
both of them date back to the 70s/80s. The first x86 processor was the Intel 8086
which was released in 1978. The origins of the first ARM machine called
Acorn Archimedes goes back to 1987. So yes, technically the ARM
architecture is newer, but looking at it from today's view, I think we could
say that both families have a long history.

So why I have decided to focus on ARM32, rather than the X86? Mainly due to
the fact that ARM is a *Reduced Instruction Set Computer (RISC)* representant,
as opposed to the *Complex Instruction Set Computer (CISC)*. The former is
(as a name suggests) supposedly less complex. Whether it is true or not I myself
cannot tell because I find many similarities between both and I use a very
limited set of tools that is offered by both architectures.

## Basic building blocks

Let's take a look at the "Hello world!" in ARM32 assembly.

```arm32
/* Hello-world program.     Print "Hello, assembly!" and exit with code 0. */
.data
    hello:
        .string "Hello, assembly!\n"

.text
    .global main
    main:
        push {ip, lr}
        ldr r0, =hello
        bl printf

        mov r0, #2
        mov r0, r1
        sub r0, r0, r1

        pop {ip, lr}
        bx lr
```

### Directives

**Directives** are used to indicate some structural information to the
assembler. They begin with a dot. In the above code block we can see
four direcrives being used:

- `.data` - starts a *data* segment of a program,
- `.text` - starts a *text* segment of a program,
- `.global` is like an `export` or `pub` from other languages, indicates
    that a name is known to the public,
- `.string` introduces a string value for a given label.

There are at least couple dozen directives but the most important in the
beginning are the ones mentioned above. The rest of them you can find in the
reference manual which is easy to find online.

### Labels

**Labels** are just names for a places inside our assembly program. We can
later on refer to them and conviniently access data or place that we would like
to jump in. To be precise they just indicate place in memory, nothing more,
nothing less. Labels end with a colon. Optionally the can start with a dot
like a directives and that indicates that they were generated by the compiler.
Again labels are information for the assembler or addiotnally debugger, and
do not have corresponding entity in the assembled machine code. We have two
labels in the above code:

- `hello:` - points to the start of the `"hello, assembly!\n"` string
- `main:` - points to the start the program

### Registers

**Registers** are special memory units that are used to hold temporary values,
values which we operate on at the moment. They are super fast and thus should
be used for our advantage as much as possible. Each of these memory units can
store up to a *machine word*, which in our case is 32 bits, four bytes.

ARM similarly to the X86 is a
[**load-store**](https://en.wikipedia.org/wiki/Load%E2%80%93store_architecture)
architecture which means that:

- data is loaded from memory into registers,
- then we perform operation on these registers,
- finally we store the results back in memory.

This way of computing has proved itself to be so efficient. Why you ask.
Foremost we can encode each register using just four bits of memory. Now
imagine you want to subtract two numbers from each other. If you would use
values directly from the memory you would have to use at least 64 bits
to encode just them. You also need to put the result somewhere so that is
additional 32 bits. Also, you need couple more bits to encode the operation
you want to perform on these two numbers, e.g., subtraction. There are more
things that you need to encode but you can already see where this is going.
Encoding subtraction for three registers will take much less. Three registers
can be encoded with just 12 bits. In fact, that is what we do, although we
use three different values we can encode the whole operation in one machine
word. This is a very simplified view of mine but I think it is enough to understand
why we would like to use this approach.

ARM32 provides 16 registers. Their names go from **r0** to
**r16**. There are aliases for the last 6 registers, because they have serve
special purpouses, hence we call them special-purpose registers. The
first 11 registers are called general-purpose registers. In essence we have:

- `r0`, `r1`, `r2`, `r3`, `r4`, `r5`, `r6`, `r7`, `r8`, `r9`, `r10` - general registers,
- `r11` or `fp` - frame pointer,
- `r12` or `ip` - intra-procedure scratch register,
- `r13` or `sp` - stack pointer,
- `r14` or `lr` - link register,
- `r15` or `pc` - program counter.

There is also an additional separate register called *current program status
register (CSPR)*.

I will uncover the purpose of the special registers in later posts.

### Instructions

Assembly gives us a set of mnemonics for the operations that we can perform
on the machinery level. There are a lot of these mnemonics so we will just
look at the most fundamental ones.

#### `add` and other arithmetic instructions

`add r0, r0, r1`

This instruction has three operands, the first one is a destination operand,
and the second, and the third one are added to each other.
This is a common pattern for ARM instructions, we use first operand as
a place where we want to store the result.

In the above example we add values from the `r0` and `r1` register, and store
the result in the `r0`.

There are more instructions that work the same way.

- `sub <dest>, <source1>, <source2>` - subtraction
- `mul <dest>, <source1>, <source2>` - multiplication
- `sdiv <dest>, <source1>, <source2>` - signed division
- `udiv <dest>, <source1>, <source2>` - unsigned division
- `bic <dest>, <source1>, <source2>` - bitwise clear (`<dest> = <source1> &~<source2>`)
- `and <dest>, <source1>, <source2>` - bitwise and
- `orr <dest>, <source1>, <source2>` - bitwise or
- `eor <dest>, <source1>, <source2>` - bitwise xor

#### `mov`

`mov r0, r1`

This instruction has two operands, the first one is a destination operand,
and the second one is a source operand. In essence it performs a copy
from the source register to the destination register.

##### Immediate operand

In some places, we can use immediate operands, i.e., literal values that
we put in our assembly source code.

`mov r0, #5` - will put decimal value five into the `r0` register.

Immediate operands must be placed as the last operand in the instruction:

- `add r0, r1, #5` - will work,
- `add r0, #5, r1` - will not.

There is also another restriction that an immediate operand must fit
inside a byte. There is an additional mechanism that in fact allows bigger
values
as long as it can be represented as an original 8-bit value with a shift. So
technically we can use larger values as long as they are from this restricted
set of values.

#### `bal` / `b`

This instruction jumps to a given label unconditionally.

`b label`

#### `bx`

`bx` aka *branch and exchange* makes a relative jump using a value from a
*register*.

`bx r0`

#### `bl`

`bl` aka *branch and link* makes a jump  to a label similarly to `b` and
additionly saves `pc` value into the `lr` register.

`bl label`

### Performing instructions conditionally

We can perform operations conditionally by adding a **condition code** at
the end of the regular instruction name. But before that, we need to set
the `CSPR` register with a result of some comparison.

We compare value using the `cmp <lhs> <rhs>` instruction. `cmp r0, r1` would
compare both registers and set the `CSPR` register accordingly. After that
we can use an instruction with the condition code like `moveq r0, r1` which
performs the mov only if the previous comparison shows that `r0` and `r1` were
equal.

```arm32
cmp r0, r1 // sets CSPR
addeq r0, r0, r1 // if(CSPR==EQ) {r0 = r0 + r1}
```

There are following condition codes:

- `eq` - equal, signed/unsigned,
- `ne` - not equal, signed/unsigned,
- `gt` - greater, signed/unsigned,
- `ge` - greater than or equal, signed,
- `lt` - less than, signed,
- `le` - less than or equal, signed,
- `hi` - greater than, unsigned,
- `hs` - greater than or equal, unsigned,
- `lo` - less than , unsigned,
- `ls` - less than or equal, unsigned,
- `al` - always, signed/unsigned.

#### `ldr` loading from memory

```arm32
ldr <dest> <source>`
```

`ldr` is used to load data from memory. We can load address of a word using
a label.

```arm32
ldr r0, =label
```

We can also load a value from an address, but this address must be already in
a register.

```arm32
ldr r1, =label // r1 will store an address
ldr r0, [r1] // r0 will store a value at this address
```

More over we can load value from an adress with an offset. Offset
can be an immediate operand or register.

```arm32
ldr r1, =some_array // r1 will store an address
ldr r0, [r1, #8] // r0 will now have value from r1 + 8 bytes
```

#### `str` storing in memory

```arm32
ldr <dest> <source>`
```

`str` stores data in memory. This one the first operand acts as a value
that we want to store in the second operand.

```arm32
str r1, [r0] // At address which was in r0 now will be value from r1
```

```arm32
ldr r1, =some_array // r1 will have an address
str r0, [r1, #8] // at the r1 + 8 bytes address will be value from r0
```

Okay these are the basics, there are a couple more instructions that we need
to know to make sense of the whole source code that I showed at the beginning
of this post but before we get to know them we need to understand some
additional mechanisms and conventions used in the world of ARM assembly.
In the next post, we will look deeper into memory, code sections, the stack,
and the heap.
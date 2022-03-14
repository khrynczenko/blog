+++
title = "32-bit ARM Assembly Basics: Part II - Segments, Memory, Stack, Heap"
date = 2021-03-31

+++

## Introduction

Let's jump right into more details regarding 32-bit ARM assembly. In this
post we will look at memory, and how it is supposed to be used, what are
the tasks for the rest of the registers, and what is the stack, and how to use
it.

## Segments

If you look at the *hello world* example from part I of this series,
you might remember two directives that we used there. They were the `.data`
and the `.text`. You might wonder what is their meaning.

`.data` section indicates that this is the start of the *data* segment of our
program. It is a segment in memory that we are allowed to **read** from,
**write** to, but **not execute**.

`.text` section indicates that this is the start of the *code* segment. It
is a segment in memory that we are allowed to **read** from, and **execute**,
but **not write** to.

This division between these two sections is due to security concerns, like
preventing executing injected code.

## Memory

When thinking about how memory is laid out in our program we think of a
continuous array. Byte after byte it is filled with some data or instructions
that make up our program. We can access this data, but there is a catch. We
can only do so for aligned words. This means that we can only access words
that are put into addresses that are divisible by four.

```
0x00 <--- We can access this 
0x01
0x02
0x03
0x04 <--- We can access this 
0x05
0x06
0x07
0x08 <--- We can access this 
```

We must remember that we address words only by the address of their first
byte. If the address of the first byte is not divisible by four we cannot
access such word.

As mentioned in the section about segments, we already know of two places where
data is stored. The data and code segments. But there are other places
where we can store data.

## The (Call) Stack

The call stack should not be thought of as a data structure. More precisely it
is just another part of memory, but it has a special purpose and should
be used in a certain way. How it is used is also related to the **calling
convention** which we will discuss in the next post.

So what for do we use the call stack? We use it to record the function call
history and store all the (not only) local variables that would not fit
into the registers.

The interesting thing is that the stack grows downwards, i.e., towards
the zero memory address. So when we push onto the stack we subtract from
the original memory address. When we pop, we add to this address.

The essential part of working with the stack is the `sp` register known as
the **stack pointer**. It should always point to the top of the stack (or the
bottom depending on how you look at it as it grows to zero).

So how do we put something on the stack? First, we must allocate memory for
new data. We do it by subtracting from the stack pointer.

```
sub sp, sp, #4
```

We have just allocated space for four bytes (machine word) on the stack. Now
we need to write data on the stack.

```
str r0, [sp]
```

That is it. We just allocated new data on the stack. In order to pop from the
stack, we do the reverse. We first get the value we want to read.

```
ldr r0, [sp]
```

After which we deallocate the memory by adding to the stack pointer.

```
add sp, sp, #4
```

That's it, but it quite cumbersome to add and subtract all by hand isn't it?
Imagine if we would like to push several values onto the stack.
That is why we have shortcuts that do exactly the same things, these shortcuts
are instructions `push` and `pop`.

```
push {r0, r1}
pop {r2, r3}
```

This is equivalent to below.

```
sub sp, sp, #8
str r0, [sp]
str r1, [sp, #4]

ldr r2, [sp]
ldr r3, [sp, #4]
add sp, sp, #8
```

This example resulted in copying values from registers `r0` and `r1` to
registers `r2` and `r3` respectively. Of course, this is just to showcase how
the `push` and `pop` instructions work. Copying data between registers this
way is inefficient.

An interesting detail is that when we use `push` and `pop` instructions
the order of registers does not matter. Lower registers will be pushed
towards lower memory addresses. So both below instructions have identical
results. For me using `gcc` requires writing the registers in ascending order.

```
push {r0, r1, r2, r3}
push {r3, r2, r1, r0}
```

That means, whenever we `pop` we pop value that was coming from the smallest
register pushed before.

```
push {r1, r2}
pop {r3, r4}  // r3 = r1, r4 = r2

```

### Stack alignnent

You probably remember that word at memory should be aligned at **four** bytes?
Well, when it comes to stack memory, there is an even bigger restriction, i.e.,
the stack pointer has to be aligned at **eight** bytes. So whenever we
push only a single word onto the stack, it becomes misaligned. That is why
in many cases when we need to push only a single word onto the stack, we
push it together with a value from some dummy register like `ip`.

```
push {r1, ip}
```

That is the reason why we pushed `lr` together with `ip` at the beginning
of our `main` function in the original example.

```
.text
    .global main
    main:
        push {ip, lr}
    ...
```

We want to push `lr` onto the stack because when calling other functions like
`printf` they are allowed to use this register for themself so we could lose
this value (more on that in the next post).
We don't need to have `ip` value on the stack but we push it anyway just to
keep things aligned.

## Heap

The **heap** is another segment of memory, also called **dynamic memory**.
We use it by requesting the operating system for allocation or deallocation. If
we are using the **libc** function these will correspond to `malloc` and
`free`. These perform system calls underneath.
The `malloc` function returns a pointer to a memory that was allocated. It
uses the value stored in `r0` for how many bytes to allocate. After allocating
it puts the aforementioned pointer into `r0`.

```
mov r0, #4
bl malloc
// r0 will now have the pointer to the allocated part of memory of four bytes
// length
```

The `free` function does not return anything, it only takes a pointer from the
`r0` register and deallocates memory at that address.

```
// assuming r0 still has the pointer
bl free
```

## What's left

Now that we understand basic instructions, what are different memory segments,
how to use stack and heap, we can dive into functions and what is *calling
convention*. It will be the last step to fully understand the original *hello
world* example. This will be explained in the part III.

+++
title = "32-bit ARM Assembly Basics: Part III - The Calling Convention"
date = 2021-04-02

+++

## Function basics

Before we jump right into specifics of calling functions, let's uncover the
purposes of the last two registers that we need.

## The Program Counter - `pc`

The **program counter** register holds the address of the instruction to be
executed. It is basically a pointer.
That means we could change the flow of the program by modifying it directly.
This is ill-advised and we should use control-flow instructions which
reduce the possibility of an error.

Every instruction modifies the `pc` register as a side effect. On every
instruction, a number of bytes that were occupied by this instruction are added
to the `pc` implicitly. Because of that our program has natural execution flow.

```arm32
mov r0, #1 /* pc += 4; r0 = 1 */
```

We can also manually cause a jump just to a given address like in the
example below.

```arm32
        mov r0, #1
        mov r0, #2
        mov r0, #3
        ldr pc, =label
        mov r0, #4
        mov r0, #5
    label:
        mov r0, #6

        /* We end up with six in r0 */
```

## The Frame Pointer - `fp`

The frame pointer points to the area of the stack before any allocation has
been made. This area is called a **stack frame** or sometimes **call
frame**.

A frame pointer is static during the procedure call, i.e., it does not
change as opposed to the stack pointer. This makes it much easier to
access local variables that we have allocated on the stack because we don't
need to take into account how `sp` was changing, we just use the `fp`.

It points at the top of the previous stack frame. So we can add to it to
access values from the parent (caller) function. We can subtract from it
to access anything that is allocated by the callee.
The thing is this is only how the frame pointer is supposed to be used.
We have to do it ourselves.

The image shows what `sp`, `fp`, and stack look like
before making a call and after making a call.

![Frame Pointer](/framepointer.png)

As you can see the frame pointer gives us easy access to our stack (for our
variables) and the caller stack (which might hold additional arguments passed
to the callee).

Okay, I omitted something. I believe that the frame pointer is most often
gonna be set to the address a little bit above the caller's stack pointer.
This is due to the fact that we need to save (push) the old frame pointer value
before we can set a new value for the frame pointer. It is easier this way
because we don't need to use registers to first push the `fp` to some register,
set `fp` to `sp`, then take value from that register and push it onto the
stack. We can avoid storing `fp` in the register if we push it onto the stack
in the first step and then assign `sp` to `fp`. We just need to change our
initial assumption that `fp` points to the beginning of the callee stack.
Instead, it will point two the beginning minus eight bytes (eight because
we need to push two values to keep alignment, and we have push `lr` anyway).

So this picture might be more realistic.

![Frame Pointer Improved](/framepointer2.png)

In our assembly it would look as follows.

```arm32
f:
    // prelude, i.e., save frame pointer and return address
    push {fp, lr}
    mov fp, sp // set our frame pointer, so have a static address
               // from which we can access other values on the stack

    // function steps

    // optionally we could call here `mov sp, fp` as  to deallocate the 
    // stack, but I think this is unnecessary if we used `pop` properly
    pop {fp, lr}
    bx lr
```

## The Calling Convetion

Now that we know how the most crucial registers are used, let us combine
all our knowledge and explain conventions behind functions in ARM32
assembly.

### Arguments and return value

The `r0`-`r3` registers are also called **arguments registers**. In these,
we can pass four word-sized parameters to the callee. If we want to pass
more arguments to the function we need to use the stack.

So if the function would need to take five arguments, before the call
we would set registers `r0`-`r3` accordingly, and push one value
onto the stack.

```arm32
mov r0, #0
mov r1, #1
mov r2, #2
mov r3, #3
mov r4, #4
push {r4, ip} // ip just to keep stack aligned
bl some_func
pop {r4, ip} // although we could just deallocate (wihout storing in register)
             // using `add sp, sp, #8`
```

As for the return value we always expect it to be place in the `r0` register.

### Register convetions

When we call some function some registers are expected to not change during
that call, i.e., the called function should not affect them, at least
from the perspective of a caller. To be precise it can change them but before
returning it should recreate their state before the call. With other
registers, the called function can do whatever it wants, and we cannot assume
anything about what is going to be left in these registers after calling
a function.

These concepts are called **call-preserved** and **call-clobbered**
respectively. Or another way: **callee-saves** and **caller-saves**. So
which registers fall under which category?

|Register|Who Saves?|
|--- | ---|
|`r0`|call-**clobbered**|
|`r1`|call-**clobbered**|
|`r2`|call-**clobbered**|
|`r3`|call-**clobbered**|
|`r4`|call-preserved|
|`r5`|call-preserved|
|`r6`|call-preserved|
|`r7`|call-preserved|
|`r8`|call-preserved|
|`r9`|call-preserved|
|`r10`|call-preserved|
|`fp`|call-preserved|
|`ip`|call-**clobbered**|
|`sp`|call-preserved|
|`lr`|call-**clobbered**|
|`pc`|saved in link register|

Remember then, if you define a new function and intend to use any register from
a **call-preserved** category, save them before usage on the stack,
and bring them back to their original state before returning.
On contrary, if you call some function and you rely on the **call-clobbered**
registers, make sure to store them, before calling this function, because it
might replace them with some garbage.

## Defining a function

To sum it up let's go back to our original *hello world* code, and go over it.

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

        mov r0, #2 // put 2 in `r0`
        mov r1, r0 // copy 2 from `r0` to `r1`
        sub r0, r0, r1 // put 0 in `r0` by subtracting `r1` from `r0`

        pop {ip, lr}
        bx lr
```

This is a very simple function. Inside we call a different function, the
`printf`. Because of that at the beginning, we push `lr` onto the stack, because
it is in the **call-clobbered** category, and we need it at the end of our
function in order to return to the caller of `main`. Next, we put
the address of our `"Hello, asembly!"` string into `r0` register. `r0` is used
as an argument for the `printf` function. Then we to some simple arithmetic
just for kicks. In the end, we restore the `lr` value and jump to the address in
it. Yay, we can read the whole file now!

Now let's define a function that takes five numbers, sums them and
prints the result.

```arm32
.data
    sum_message:
        .string "The sum is %u\n"
.text
    sum_five:
        // prelude, i.e., save frame pointer and return address
        push {fp, lr}
        mov fp, sp

        add r0, r0, r1
        add r0, r0, r2
        add r0, r0, r3
        ldr r3, [fp, #8] // We get the last argument from from the
                         // previous call stack
        add r0, r0, r3

        pop {fp, pc}

    .global main
    main:
        // prelude, i.e., save frame pointer and return address
        push {fp, lr}
        mov fp, sp // set our frame pointer, so have a static address
        // from which we can access other values on the stack

        // prepare arguments to sum up
        mov r0, #0
        mov r1, #1
        mov r2, #2
        mov r3, #3
        mov r4, #4
        push {r4, ip}

        bl sum_five 
        add sp, sp, #8 // deallocate the stack
        
        mov r1, r0 // put number to print in `r1`
        ldr r0, =sum_message // put address of the string with format to
                             // print in `r0`
        bl printf

        pop {fp, lr}
        bx lr

```

In the main function, we put the first four arguments in the `r0`-`r3`
registers.
We put the additional argument onto the stack using push. Then we call
the `sum_five` function which saves `fp` and `sp` (although we don't call
any function so this is kind of unnecessary here). We add values from the
registers and load the last argument using the frame pointer with a relative
offset of eight bytes. We sum all this in `r0` which is used to store
return values. The rest is easy, we return, we call `printf` and return
from `main`.

## Conclusions

I hope I got everything right. I know I have been
omiting some details like how arguments of a size different than word should be
passed etc., so I still don't know how much of the stuff works, I must say
that I am capable now of reading the ARM32 assembly and even writing some
of my own which makes me very happy. I would advise any other than myself
to read this series that I made because there are plenty of others much better
prepared tutorials.

I was using two books to get the basics that I laid out in this series.

- "Compiling to Assembly" by Vladimir Keleshev
- "Introduction to Compilers and Language Design" by Douglas Thain

Both these books are great and go over things in more detail. Anyway, thanks for
reading!

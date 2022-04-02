+++
title = "Two's complement and overflows"
date = 2022-04-02

+++

## Numbering systems

Before going into what is *Two's complement* and its relation to the number
overflows and underflows, let's quickly overview the ubiquitous numbering
systems.

The one natural to us humans is a base-10 (decimal) system. It's called base-10
because digits are one of ten distinct symbols and each consecutive
digit in a number has increased weight by a factor of ten.

```
124 (base-10)

|    1    |     2    |     4    |

 10^2 * 1 + 10^1 * 2 + 10^0 * 4 = 124
```

This means that in the base-2 system (binary) we use two distinct symbols,
0s and 1s, and the weight of each consecutive digits increases by a factor of
two.

```
1001 (base-2) = 9 (base-10)

|    1   |     0   |     0   |     1   |

 2^3 * 1 + 2^2 * 0 + 2^1 * 0 + 2^0 * 1 = 9
```

And for the base-16 system (hexadecimal) we use 16 distinct symbols,
from 0-9 and A-F, and the weight of each consecutive digit increases by a
factor of 16.

## Representing signed numbers

Two's complement is a method for representing negative numbers. For N-bit
number the two's complement is a number with respect to 2^N that when added
to the original number gives us 2^N.

It means that for the 4-bit number 0101 the two's complement is 1011, because
`0101 + 1011 = 10000` and `10000 = 2^4`. We can already notice a nice property
that if we would restrict the number of bits, i.e., not allow to add
the new bit, the result would be `0101 + 1011 = 0000`. This means that
whenever we add a two's complement to a number the result is 0. That is why
two's complement works. It turns a positive number into the same negative
number. These must sum to zero.

In essence, if we want to create a negative number out of positive we:

- complement by flipping each bit in a number,
- add 1 to this number.

```
1) 0101 - positive number
2) 1010 - flip the bits
3) 1011 - add one, as a result obtain a negative number

```

One thing to notice here is that a negative number always starts with a
**one** and a positive number always starts with a **zero**.

## Explaining overflow/underflow behavior

### Unsigned numbers

In C++ unsigned numbers have well defined behavior in a way that the overflow
for them causes wrapping around.

```c++
std::numeric_limits<unsigned int>::max() + 1 = 0;
std::numeric_limits<unsigned int>::max() + 2 = 1;
``` 

The standard defines the result as `mod (2^N)` on the result of such
addition.

```
Assuming we are working with 8-bit number (MAX = 255).

255 + 1 (overflow) = 256 mod (2^8) = 0
255 + 2 (overflow) = 257 mod (2^8) = 1
```

And this makes total sense if we try do do such operation in binary. Let's try
that with a 4-bit unsigned number for which the maximum value is 16.

```
  1111 (15)
+ 0001 (1)
= 0000 (0)
```

```
  1111 (15)
+ 0010 (2)
= 0001 (1)
```

For underflow it works they same just the other way around.

```
  0000 (0)
+ 0001 (1)
= 1110 (14) (flip)
```

```
  0000 (0)
+ 0010 (2)
= 1101 (13) (flip)
```

## Signed numbers

Why does the standard not dictate a behavior for signed integers as well?
It is because Two's complement is not the only way of working with singed
integers even though it might be the most ubiquitous one. Because
of that, the overflow for the signed numbers is undefined behavior.

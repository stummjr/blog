---
title: "How Does CPython Multiply Big Numbers?"
author: "Valdir Stumm Jr"
tags: ["python", "internals"]
date: 2021-10-04T20:21:10-03:00
author: Valdir Stumm Jr
draft: false
---

I am used to write code that multiplies numbers several times a week. Usually when I do that, I don't think much about the operation itself or how the machine will execute it.

But if we start thinking about it, how in hell can the [CPython](https://github.com/python/cpython) interpreter multiply numbers as large as the ones below? The CPU definitely does not support huge numbers like that out of the box, so how does it work?

```python
>>> 92982374592874395723984756872342342234 * 670878370598623450872390483452435
```

The answer is both simple and complex. 😃

First of all, we have to acknowledge the fact that the numbers that we can represent in Python are way larger than the numbers that a modern CPU can. For example 10^100 is huuuuge but Python can handle it:

```python
>>> 10 ** 100
10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

To support that, CPython implements a long object (any integer number, basically) with an [array of digits](https://github.com/python/cpython/blob/bb3e0c240bc60fe08d332ff5955d54197f79751c/Include/longintrepr.h#L85-L88):

```c
struct _longobject {
    PyObject_VAR_HEAD
    digit ob_digit[1];
};
```

I don't know about you, but the first thing that comes to my mind when I think on how to multiply two numbers that are broken down digit by digit is to use that good old algorithm that we learn in grade school, creatively called [Grade-School Multiplication](https://en.wikipedia.org/wiki/Multiplication_algorithm#Long_multiplication).

That is, given that I want to multiply 232 * 23, I can do something along these lines:

```
  232
  *23
 ----
  696
+464
-----
 5336
```

I am pretty sure you did that a bunch of times in your life and, believe it or not, CPython uses the very same multiplication algorithm _most of the time_.


## Diving Into CPython Multiplication
Before multiplying two numbers, CPython checks if at least one of the numbers is small enough to be handled efficiently with the grade-school algorithm.

Simply put, CPython will use the grade-school algorithm (implemented in [`x_mul`](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c#L3197)) to multiply two operands when at least one of these operands is less than 71 digits long (in **base 2^30**), as we can see [here](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c#L3356-L3363):

```c
/* Use gradeschool math when either number is too small. */
i = a == b ? KARATSUBA_SQUARE_CUTOFF : KARATSUBA_CUTOFF;
if (asize <= i) {
    if (asize == 0)
        return (PyLongObject *)PyLong_FromLong(0);
    else
        return x_mul(a, b);
}
```

### Base 2^30?
That's right, CPython internally represents the numbers as an array of `uint32_t`, where 30 out of 32 bits of each element are used for the actual value of that digit. And CPython will employ the grade-school algorithm when any of the operands have less than 71 base 2^30 digits.

A 71 digits number, where each digit can represent up to 2^30-1, is quite a huge number! Think about it, a 71 **decimal** digits number is a humongous number already, even though each digit can represent only up to 9. Now think about a number broken down in 71 parts, where each part can represent up to 1073741823.


### Optimizations
In order to be fast, CPython employs many optimizations for special cases. If you look at the snippet above, you will notice that if one of the numbers is zero, CPython doesn't even try to multiply them and [returns 0 immediately](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c#L3360).

CPython also optimizes the multiplication of identical numbers (aka squaring) by using a separate algorithm for that case, as you can see in the [`x_mul`](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c#L3209-L3252) function implementation.


### What if the numbers are too big?
CPython defines `KARATSUBA_CUTOFF` as 70 and the reason for the constant name is that in case both operands are too big (more than 70 digits long), CPython will employ the [Karatsuba multiplication algorithm](https://en.wikipedia.org/wiki/Karatsuba_algorithm) (implemented in [`k_mul`](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c#L3322)), which is significantly faster than the traditional algorithm.

Given that Karatsuba is a recursive algorithm, `k_mul` recursively breaks the numbers in sub-parts and before multiplying them, it checks again if they are still big enough to use Karatsuba, otherwise it applies the grade-school algorithm. Now, I am not an expert on Karatsuba to explain it in any straightforward way, but [this video](https://www.youtube.com/watch?v=JCbZayFr9RE) is super helpful in case you want to understand it.


## Wrapping up

This is not supposed to be a super deep dive into CPython, but I just wanted to show how fascinating it is to dive a tad bit on the stuff that we take for granted. Take multiplication for example: how often do we think that a simple multiplication in our program may actually be a O(N^2) operation? Even simpler operations, like adding two numbers, can be much more complex than we think when we use high level languages that support big numbers.

And please don't get me wrong, I really love the fact that I can easily multiply two huge numbers in Python without caring about overflows. However, it's also great to know about what actually happens under the hood.

## References
- [Python internals: Arbitrary-precision integer implementation](https://rushter.com/blog/python-integer-implementation/)
- [longobject.c](https://github.com/python/cpython/blob/ef9e22b253253615098d22cb49141a2a1024ee3c/Objects/longobject.c)

---
title: "Python 3 rounding oddities"
author: "Valdir Stumm Jr"
tags: ["python", "rounding"]
date: 2018-08-28T15:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

Rounding a decimal number with Python 3 is as simple as invoking the [`round()`](https://docs.python.org/3/library/functions.html#round) builtin:

```python
>>> round(1.2)
1
>>> round(1.8)
2
```

We can also pass an extra parameter called `ndigits`, which defines the precision we want in the result. Such parameter defaults to 0, but we can pass anything:

```python
>>> round(1.847, ndigits=2)
1.85
>>> round(1.847, ndigits=1)
1.8
```

And what happens when we want to round a number like 1.5? Will it round it up or down? Let’s check:

```python
>>> round(1.5)
2
```

It seems that it rounds up. Let’s check some other numbers to confirm:

```python
>>> round(2.5)
2
```

Uh, now it went down! Let’s check some more:


```python
>>> round(3.5)
4
>>> round(4.5)
4
>>> round(5.5)
6
```

![](/img/posts/michael-scott-eli5.png)

Calm down, there’s an explanation for this. In Python 3, `round()` works like this:

> Round to the closest number.
> If there’s a tie, round to the closest even number.

Now it makes sense. If we check the examples above, we’ll see that the rounding was always made to the closest even number:



```python
>>> round(3.5)
4
>>> round(4.5)
4
>>> round(5.5)
6
```

# What about Python 2?

Python 2 is quite different. When there’s a tie, the rounding is always made upwards in case the numbers are positive:



```python
>>> round(1.5)
2.0
>>> round(2.5)
3.0
```

And downwards, when the numbers are negative:


```python
>>> round(-1.5)
-2.0
>>> round(-2.5)
-3.0
```

# Why the hell did Python 3 changed it?

The goal is to take the **bias** out of the rounding operations.

Imagine a bank where all the roundings are done upwards. By the end of the day, the bank earning report will show a value that is higher than what the bank actually earned. That’s what happens on **Python 2**:



```python
>>> # Python 2
>>> values = [1.5, 2.5, 3.5, 4.5]
>>> sum(values)
12.0
>>> sum(round(v) for v in values)
14.0
```
Using Python 3’s `round()`, the rounded values tend to be amortized, because half of them round upwards and half of them round downwards, given that half the numbers are even and the other half are odd. Check the same code, but now running on **Python 3**:


```python
>>> # Python 3
>>> values = [1.5, 2.5, 3.5, 4.5]
>>> sum(values)
12.0
>>> sum(round(v) for v in values)
12
```

This is no Python 3’s inovation. In fact, this kind of rounding is quite old and even has a proper name: [Bankers Rounding](http://wiki.c2.com/?BankersRounding).


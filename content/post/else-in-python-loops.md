---
title: "The curious case of the else in Python loops"
date: 2018-09-05T15:42:35-03:00
tags: [python, programming, loops]
author: Valdir Stumm Jr
draft: false
---

One of the first things to stand out when I was starting with Python was the `else` clause. I guess everyone knows the normal usage of such clauses in any programming language, which is to define an alternate path for the `if` condition. Oddly enough, in Python we can add `else` clauses in loop constructions, such as `for` and `while`.

For example, this is valid Python:

```python
for number in some_sequence:
    if is_the_magic_number(number):
        print('found the magic number')
        break
else:
    print('magic number not found')
```

Notice how the `else` is aligned with the `for` and not with the `if`. What this means is that commands inside the `else` block will be executed if, and only if, the loop was not finished by a `break`. The same is true for `while` loops.

I must admit that I’ve always had some trouble to remember the meaning of an `else` in loops, specially because I don’t see them very often (and I’m grateful for that). But, at some day I was watching [Raymond Hettinger’s Transforming Code into Beautiful, Idiomatic Python](https://youtu.be/OSGv2VnC0go) talk where he brilliantly says something like this at some point:

> Why don’t you call the else in loops as ‘nobreak’?

That’s all I needed to not forget the meaning anymore. 🙂

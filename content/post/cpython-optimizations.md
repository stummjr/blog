---
title: "CPython Optimizations"
author: "Valdir Stumm Jr"
tags: ["python", "internals"]
date: 2020-06-19T23:19:13-03:00
author: Valdir Stumm Jr
draft: false
---

[CPython](https://github.com/python/cpython/) is the reference implementation of the Python language. While there are several other implementations, CPython is by far the most popular one, as it comes bundled in most of the operating systems these days.

While it is not particularly well-known as a high-performance interpreter among its peers, it does some very interesting optimizations to speed itself and Python programs up. I am pretty curious about things like these and the rationale behind them, even though I am very far from being an expert on this subject matter. What follows is the result of some experimentation, myself reading CPython's source code and reading documentation/blog posts.


## Caching Small Integers
CPython caches small integers from -5 to 256 in an internal array during its initialization. That means that every time the interpreter itself or your Python program needs to use a number in that range, CPython won't have to allocate space and create a brand new object for that. It will just return a reference to an existing object.

Check this out:
```python
>>> x = -5
>>> y = -5
>>> x is y
True
>>> x = -6
>>> y = -6
>>> x is y
False
```
As you can see, `x` and `y` are initially pointing to the exact same object because `-5` is one of the small integers cached by CPython. With `-6`, however, we have a different outcome. Now `x` and `y` point to different objects, even though they have the very same value.

As you can see, these singletons are used even when computing the result of arithmetic expressions:

```python
>>> x = 3 * 2
>>> y = 5 + 1
>>> x is y
True
```

CPython caches these small numbers because they are very often used in arithmetic operations, as boundaries in loops and as results of small computations. If CPython had to allocate memory and create a new object for each instance of these numbers, it would spend a bunch of time and space doing so.

## String Interning
[String Interning](https://en.wikipedia.org/wiki/String_interning) is an optimization technique implemented by many modern compilers and interpreters. It consists of creating and storing only a single instance of a given string, rather than multiple. It makes a lot of sense, if you think about it, as strings are immutable objects in Python.

In practical terms it means that some (not all, as we'll see soon) strings will be "cached" by the Python interpreter. Here we can see that CPython creates a single string object to hold the "hey" string:
```python
>>> x = 'hey'
>>> y = 'hey'
>>> x is y
True
```
However, CPython does not intern each and every string. Check out how the strings below were not cached/interned:

```python
>>> x = 'hey!'
>>> y = 'hey!'
>>> x is y
False
```

That's because CPython won't intern strings that are not composed exclusively by ASCII letters, numbers or underscores (see [codeobject.c](https://github.com/python/cpython/blob/314858e2763e76e77029ea0b691d749c32939087/Objects/codeobject.c#L24-L40)). With this rule CPython "ensures" that strings that look like valid Python identifiers will be interned. This way, function/method/variable names will end up interned and thus lookups will be faster during bytecode execution.

Single-character strings will also be reused. CPython checks if the string has a single digit before allocating a new object. If it has, it will just return a reference to an already existing object that represents that character (see [unicodeobject.c](https://github.com/python/cpython/blob/eb0d5c38de7f970d8cd8524f4163d831c7720f51/Objects/unicodeobject.c#L2321-L2338)).

Check this out:

```python
>>> x = 'á'
>>> y = 'á'
>>> x is y
True
```

It's not entirely clear to me in which cases exactly a string is interned. I've read conflicting information on the subject and my experiments were somewhat confusing.

### Forcing CPython to Intern a String
If for some reason you want to force some strings to be interned by CPython, you can call the `sys.intern` function passing a string as its argument. This may be helpful if you have huge strings that you'll need to reuse often and that would not be automatically interned by CPython. For example:

```python
>>> x = 'hey!'
>>> y = 'hey!'
>>> x is y
False
>>> x = sys.intern('hey!')
>>> y = sys.intern('hey!')
>>> x is y
True
```

## Constant Folding
[Constant folding](https://en.wikipedia.org/wiki/Constant_folding) is another technique often employed by compilers and interpreters. It consists of precomputing in compile time the expressions that have no runtime dependencies. For example, it is quite common to find definitions like this in Python programs:

```python
KILOBYTE_IN_BITS = 8 * 1024
```
People usually do that so that other people reading their code can get a better grasp on what an otherwise "magic" constant actually represents.

The CPython interpreter folds that expression into a constant while compiling the source code (`.py`) into bytecode (`.pyc`). In practical terms, it means that the resulting bytecode won't contain the `8 * 1024` expression, but the `8192` constant instead. We are basically trading run time for compile time. Imagine if `KILOBYTE_IN_BITS` was defined in a function that is called thousands of times during a program execution. We'd have thousands of multiplications, all happening in runtime.

### Digging a bit more
Before we start digging, let me show you how we can verify if a given expression is being folded into a constant or not. An easy way to do that is using the [`dis`](https://docs.python.org/3/library/dis.html) module do disassemble a Python function into its bytecode representation.

```python
>>> from dis import dis
>>> def func():
        KILOBYTE_IN_BITS = 8 * 1024
>>> dis(func)
  3           0 LOAD_CONST               1 (8192)
              2 STORE_FAST               0 (KILOBYTE_IN_BITS)
              4 LOAD_CONST               0 (None)
              6 RETURN_VALUE
```

As you can see in the bytecode outputted by the `dis` call above, there's no multiplication being done and we have `8192` as a constant value instead.

### Not just for numbers
Constant folding is not used exclusively for arithmetic expressions. Expressions that compute strings are also candidates for that optimization. Check this out:

```python
>>> def func():
        s = '-' * 10
>>> dis(func)
  2           0 LOAD_CONST               1 ('----------')
              2 STORE_FAST               0 (s)
              4 LOAD_CONST               0 (None)
              6 RETURN_VALUE
```

See how the bytecode contains the computed version of the string `s` already. Not all string operations will be folded, though:

```python
>>> def func():
        y = '-' * 5000

>>> dis(func)
  2           0 LOAD_CONST               1 ('-')
              2 LOAD_CONST               2 (5000)
              4 BINARY_MULTIPLY
              6 STORE_FAST               0 (y)
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

As you can see in the bytecode above, we have a `BINARY_MULTIPLY` instruction that will take place in runtime.

#### Large strings won't be folded
My first hypothesis when I saw the output above was that it was something related to the string size. It makes a ton of sense for CPython to not expand expressions like `'-' * 10000000` into constants, as that would generate bigger `.pyc` files. But I got curious on what's CPython threshold for that. 

To find that out, I wrote a quick and dirty function to check if a given expression would be folded or not:

```python
def folds(string, size):
    c = compile(f"'{string}' * {size}", 'dummyfile', 'eval')
    # in case the constant has not been folded, `co_consts` will be (string, size) 
    # in case it was folded, it will be represented by a tuple with the resulting string
    return c.co_consts[0] != string
```

That function allowed me to find out that **CPython won't fold strings bigger than 4096 characters into constants**. Check this out:

```python
>>> folds('-', 4096)
True
>>> folds('-', 4097)
False
>>> folds('--', 2048)
True
>>> folds('--', 2049)
False
```

### What about Constant Propagation?
As far as I can tell, CPython **does not do** [Constant Propagation](https://en.wikipedia.org/wiki/Constant_folding#Constant_propagation). That is, it does not replace variables whose values are known in compile time by their actual values. Check this out:

```python
>>> def func():
        KILOBYTE_IN_BITS = 8 * 1024
        MEGABYTE_IN_BITS = KILOBYTE_IN_BITS * 1024
>>> dis(func)
  2           0 LOAD_CONST               1 (8192)
              2 STORE_FAST               0 (KILOBYTE_IN_BITS)

  3           4 LOAD_FAST                0 (KILOBYTE_IN_BITS)
              6 LOAD_CONST               2 (1024)
              8 BINARY_MULTIPLY
             10 STORE_FAST               1 (MEGABYTE_IN_BITS)
             12 LOAD_CONST               0 (None)
             14 RETURN_VALUE
```

If CPython did it, we'd not see the `BINARY_MULTIPLY` instruction.

## Dead code elimination
CPython also eliminates dead code, such as unreachable statements. In the example below, the first `print` will never be reached, so no bytecode is generated for that branch:

```python
>>> def func():
      if False:
          print("hello")
      print("hey!")

>>> dis.dis(func)
  4           0 LOAD_GLOBAL              0 (print)
              2 LOAD_CONST               1 ('hey!')
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

Any code added after the `return` statement will also be eliminated:

```python
>>> def func():
        return "hey"
        s = "what am I doing here?"

>>> dis(func)
  2           0 LOAD_CONST               1 ('hey')
              2 RETURN_VALUE
```

## Wrapping up

Keep in mind that the optimizations listed in this blog post are specific to the CPython interpreter. Other interpreters like PyPy employ additional techniques to speed themselves up, but they were not covered in this blog post.

Optimizations like this are not part of the language specification, obviously. Each interpreter is free to implement as many optimizations as they want, as long as the semantics stays the same. One important takeaway is that the correctness of your code should never rely on any of these optimizations, as they are implementation details and could be dropped any time.


## Read more
- [The Internals of Python String Interning](http://guilload.com/python-string-interning/)
- [Python Caches Integers](https://arpitbhayani.me/blogs/python-caches-integers)

---
title: "Drop Duplicates from a List in Order"
date: 2018-08-27T15:42:35-03:00
tags: [python, programming, list]
author: Valdir Stumm Jr
draft: false
---

Let’s say you have a list containing all the URLs extracted from a web page and you want to get rid of duplicate URLs.

The most common way of achieving that might be building a set from that list, given that such operation automatically drops the duplicates. Something like:

```python
>>> urls = [
    'http://api.example.com/b',
    'http://api.example.com/a',
    'http://api.example.com/c',
    'http://api.example.com/b'
]
>>> set(urls)
{'http://api.example.com/a',
 'http://api.example.com/b',
 'http://api.example.com/c'}
```

The problem is that we just lost the original order of the list.

A good way to maintain the original order of the elements after removing the duplicates is by using this trick with [`collections.OrderedDict`](https://docs.python.org/3/library/collections.html#collections.OrderedDict):

```python
>>> from collections import OrderedDict
>>> list(OrderedDict.fromkeys(urls).keys())
['http://api.example.com/b',
 'http://api.example.com/a',
 'http://api.example.com/c']
```

Cool, huh? Now let’s dig into details to understand what the code above does.

`OrderedDict` is like a traditional Python `dict` with a (not so) slight difference: `OrderedDict` keeps the elements’ insertion order internally. This way, when we iterate over such an object, it will return its elements in the order in which they’ve been inserted.

Now, let’s break down the operations to understand what’s going on:

```python
>>> odict = OrderedDict.fromkeys(urls)
```

The [`fromkeys()`](https://docs.python.org/3/library/collections.html#collections.Counter.fromkeys) method creates a dictionary using the values passed as its first parameters as the keys and the second parameter as its values (or `None` if we pass nothing, as we did).

As a result we get:

```python
>>> odict
OrderedDict([('http://api.example.com/b', None),
             ('http://api.example.com/a', None),
             ('http://api.example.com/c', None)])
```

Now that we have a dictionary with the URLs as the keys, we can call the `keys()` method to get only a sequence containing the URLs:

```python
>>> list(odict.keys())
['http://api.example.com/b',
 'http://api.example.com/a',
 'http://api.example.com/c']
```

Easy like that. 😀
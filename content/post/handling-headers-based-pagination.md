---
title: "Handling Headers-Based Pagination on APIs with Python"
author: "Valdir Stumm Jr"
tags: ["python", "til", "tips"]
date: 2020-07-12T15:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

The other day I was building a Python script to fetch some information about all my repositories on GitHub.
Their API is pretty straightforward and [well documented](https://developer.github.com/v3/).

Fetching my repos was as simple as:

```python
>>> import requests
>>> response = requests.get("https://api.github.com/users/stummjr/repos")
```

The thing is that such request gave me a list with all my repos on it. No info on pagination whatsoever on the response **body**. After a bit of research, I found out that [GitHub APIs take advantage of the Link header](https://developer.github.com/v3/#link-header) to expose several pagination options. Given that I hadn't seen such a header before, I went ahead and inspected it:

```python
>>> response.headers["link"]
'<https://api.github.com/user/1170435/repos?page=2>; rel="next", <https://api.github.com/user/1170435/repos?page=2>; rel="last"'
```

Cool! It's not just giving me a link to the next page, but also to the last one. Nice!

But wait, that's a string! What am I supposed to do with that? Write a small parser to extract such headers? Shouldn't be hard to do so, but there must be a better way.

Of course there is! The awesome requests library never let me down and this time was no exception. It parses such headers and exposes the pagination info via the [`Response.links`](https://2.python-requests.org/en/master/user/advanced/#link-headers) attribute:

```python
>>> response.links
{
  'next': {
    'url': 'https://api.github.com/user/1170435/repos?page=2',
    'rel': 'next'
  },
  'last': {
    'url': 'https://api.github.com/user/1170435/repos?page=2',
    'rel': 'last'
  }
}
>>> response.links["next"]
{'url': 'https://api.github.com/user/1170435/repos?page=2', 'rel': 'next'}
>>> response.links["last"]
{'url': 'https://api.github.com/user/1170435/repos?page=2', 'rel': 'last'}
```

Simple like that. 🙂

If you want to learn more about this standard, check out the [IETF's RFC8288](https://tools.ietf.org/html/rfc8288).
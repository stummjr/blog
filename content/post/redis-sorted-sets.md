---
title: "Counting With Redis Sorted Sets"
author: "Valdir Stumm Jr"
tags: ["python", "redis", "data structures"]
date: 2023-01-18T13:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

I've used Redis as a cache layer multiple times in my career.
Most of the times, all I needed was a fast storage where I could store pre-computed JSON data and retrieve it by key.
Redis was just perfect for that. Lightweight, easy to use, good docs, widely available in Cloud providers, etc.

A couple weeks ago, though, I was faced with a different problem that helped me learn Redis a little more in depth.

The general problem is a quite common one: keeping track of the number of hits to a given resource and listing the top resources,
in an efficient way. A problem like this may take many forms: top songs in a music streaming service, a leaderboard in a video game, etc.


## My first idea
There are many ways to do that, and I was initially inclined to use a simple counter in Redis, using the resource name
as the key. So, every time there was a hit to a resource, I could just increment a counter:

```python
>>> redis_client = redis.Redis(host="localhost", port=6379, encoding="utf-8", decode_responses=True)
>>> redis_client.incr("/foo/bar.png")
1
>>> redis_client.incr("/foo/bar.png")
2
>>> redis_client.incr("/api/foo/")
1
>>> redis_client.get("/foo/bar.png")
'2'
```

While that works well for keeping track of the hits on each resource, it would not be very efficient for retrieving
the most-accessed resources. Redis would not give me the results sorted by counter and I'd have to do so in the
application level, which would be problematic given that there could be many thousands of resources to keep track of.


## A better approach
**_"Redis must have a way to do that natively..." (scratching my head)_**

So I started looking into Redis (excellent) documentation on its data types and found a page describing
[Sorted Sets](https://redis.io/docs/data-types/sorted-sets/). Here's how they are described:

> A Redis sorted set is a collection of unique strings (members) ordered by an associated score.

**Bingo!** I can use the resource names as keys inside the set and increment the associated score every
time the resource is hit. While that sounds similar to what I was doing with my first approach, in this
case I can use sorted sets' builtin [`ZRANGE`](https://redis.io/commands/zrange/) function to
retrieve the elements in a range *sorted by score*. Neat, huh?

This is how we can increment the counter for the resources:

```python
>>> redis_client.zadd("myapp:hits", {"/foo/bar.png": 1}, incr=True)
1.0
>>> redis_client.zadd("myapp:hits", {"/foo/bar.png": 1}, incr=True)
2.0
>>> redis_client.zadd("myapp:hits", {"/api/foo/": 1}, incr=True)
1.0
```

That is, in order to increment the counter for a given resource in the sorted set, we provided the key
to access the whole set in Redis (`myapp:hits` in this case), a mapping with the resource name mapped to
the amount that we want to increment its score by (`1`), and finally the `incr` flag as `True`.

Now that we know how to store the resources and their number of hits in a set, the next thing is to find a way
to retrieve the top `n` resources. For that we'll use the aforementioned `ZRANGE` function, like this:

```python
>>> n = 10
>>> redis_client.zrange("myapp:hits", 0, n-1, withscores=True, desc=True, score_cast_func=int)
[('/foo/bar.png', 2), ('/api/foo/', 1)]
```

Here we're basically telling `zrange` to retrieve the top `n` elements in the set (0...n-1), including their scores,
in descending order, and finally casting the scores from floats to ints.

Once I figured that out, all I had to to was to setup a _before-request_ hook in my web app to increment the counter
when the resources are hit and create a simple endpoint to return the top resources.

As you can see, Redis handled most of the complexity for me and I ended up with a very simple solution in the application side.
All I had to do was to spend some time reading Redis documentation.

I'm sure there are better ways to do that, but this seemed like a pretty good first implementation and a good exercise on
diving a little more into Redis.

## Wrapping up
If you're interested in learning more about Redis, I suggest you to read the docs on its data types: https://redis.io/docs/data-types/.
It's very concise and it perhaps will add some resources to your toolbelt that can be useful the next time you're facing a problem that
can be solved via Redis.

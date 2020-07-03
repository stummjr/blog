---
title: "Writing Scrapy Spiders in 2020"
author: "Valdir Stumm Jr"
tags: ["python", "scrapy"]
date: 2020-05-03T15:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

I am a huge fan of [Scrapy](https://scrapy.org/) and I've used it extensively for 3+ 
wonderful years working at Scrapinghub, the company behind this framework.

It's been one and a half year since I used it for the last time, but last week I had to build a 
spider for a personal project. To my surprise, I am not just rusty but pretty outdated in
terms of the new shiny features of Scrapy.

To help other people in the same situation as myself, I am going to go through some of the
main changes since version 1.5.2 (the last one I  had used) and 2.1.0 (the current one).

# Following links in 2020
Back when I used Scrapy in a daily basis, this is how I'd make my spider follow through
links found on the page:

```python
links = response.css("a.entry-link::attr(href)").extract()
for link in links:
    yield scrapy.Request(url=response.urljoin(link), callback=self.parse_blog_post)
```

In 2020, I can rewrite this snippet using [`Response.follow`](https://docs.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response.follow):

```python
links = response.css("a.entry-link")
for link in links:
    yield response.follow(link, callback=self.parse_blog_post)
```

Notice how I didn't even have to extract the link as a string. That is pretty cool.

But now that we're all using Python 3 (*wait, aren't you yet?*), we can just do it like
this:

```python
links = response.css("a.entry-link")
yield from response.follow_all(links, callback=self.parse_blog_post)
```

*Neat, huh?*

# Extracting data in 2020

## get() and getall()
There are tons of docs on Scrapy around the web showing you how to scrape data 
using the `extract` and `extract_first` selector methods. This is how I used to write the
data extraction side of a spider using them:

```python
def parse_blog_post(self, response):
    yield {
        "title": response.css(".post-title::text").extract_first(),
        "author": response.css(".entry-author::text").extract_first(),
        "tags": response.css(".tag::text").extract(),
    }
```

This isn't a big change, but now we can use `getall` and `get` instead of `extract` and `extract_first`:

```python
def parse_blog_post(self, response):
    yield {
        "title": response.css(".post-title::text").get(),
        "author": response.css(".entry-author::text").get(),
        "tags": response.css(".tag::text").getall(),
    }
```

Looks cleaner and easier to understand to me.


## The new attrib dict
A quite common case that I had back in the days was to have to extract multiple attributes
from a single node. For example, let's say I want to extract both the `alt` and the `src` 
attributes from this `img`:

```html
<img alt="Super cool" src="/img/supercool.jpg" />
```

Back in the days, I'd do something like this:

```python
yield {
    "url": response.css(".header img::attr(src)").extract_first(),
    "description": response.css(".header img::attr(alt)").extract_first(),
    "size": response.css(".header img::attr(sizes)").extract_first(),
}
```

There's more repetition in this snippet that a person should be allowed to write in their life.

In 2020, I can avoid such repetition by using the
[`attrib`](https://docs.scrapy.org/en/latest/topics/selectors.html#selecting-element-attributes)
dict available in `Selector` and `SelectorList` objects:

```python
img_sel = response.css(".header img")
yield {
    "url": img_sel.attrib["src"],
    "description": img_sel.attrib["alt"],
    "size": img_sel.attrib["sizes"],
}
```

*Pretty sick!*

I remember doing ugly hacks using string interpolation in the selectors to avoid repetition.
This is so much better!


# Passing callback arguments in 2020
Every now and then, I'd have to pass some data from one callback to another so that they
could share some state. Back then, I'd pass it via the `meta` parameter in `Request` 
objects.

While that worked pretty well, it wasn't that great for the spider readability, as you
couldn't tell a callback's interface just by looking at its signature. Check it out:

```python
def parse_blog_post(self, response):
    ...
    for link in links:
        yield scrapy.Request(
            link,
            meta={"author": author, "post_date": post_date},
            callback=self.parse_full_blog_post,
        )

def parse_full_blog_post(self, response):
    author = response.meta["author"]
    post_date = response.meta["post_date"]
    ...
```

*Cool, but not so cool.* Now we can use `cb_kwargs` and declare the parameters in the 
callback's signature instead:

```python
def parse_blog_post(self, response):
    ...
    yield from response.follow_all(
        links,
        cb_kwargs={"author": author, "post_date": post_date},
        callback=self.parse_full_blog_post,
    )

def parse_full_blog_post(self, response, author, post_date):
    ...
```

That's much better. Now my callback has a proper signature and the spider will fail in case I
don't provide the proper callback arguments.


# Wrapping up
I am sure there are tons of new features on Scrapy that would deserve each a blog post.
As I am not a power user anymore, the changes that I listed above are the ones that
impact me the most.

If you're a Scrapy user, please start writing Scrapy as if you were in 2020 and spread
these new features in your circles.

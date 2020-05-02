---
title: "Debugging Python with pudb"
date: 2020-05-01T15:42:35-03:00
tags: [python, debugging]
draft: false
---


[**Pudb**](https://github.com/inducer/pudb) is, in my opinion, the most underrated Python package out there. I know this is a bold statement, but that’s how I feel about it. It helped me so much in a daily basis for so many years and I still feel like not too many people know about it.

# Debugging in Python
There are several good debuggers for Python. I know a ton of people that use [pdb](https://docs.python.org/3/library/pdb.html), [ipdb](https://github.com/gotcha/ipdb), VSCode/PyCharm embedded debuggers, among others.

I tried most of them and they are actually pretty good, but none of them made me feel at home as **pudb** did. Maybe it’s the *Turbo Pascal-esque* UI that brings me 20 years back in time. Maybe it’s the ability to debug stuff visually in a terminal. I don’t know.

# Meet pudb
I was going to write a bunch of stuff here, but then I realized that it makes a lot more sense to just demo pudb’s main features, so here we go:


{{< youtube bJYkCWPs_UU>}}


# How to install it
Pudb is a third-party package, so you’ll need to install it first:

```bash
$ pip install pudb
```

# How to Launch it
Launching pudb is similar to launching other python debuggers such as pdb and ipdb. All you have to do is to drop the line below wherever you want the execution of your program to pause:

```python
import pudb; pudb.set_trace()
```

You can also launch your program with pudb via command line:

```bash
$ python -m pudb your_script.py
```

# Using it on Docker
Fortunately, pudb supports [remote debugging](https://documen.tician.de/pudb/starting.html#remote-debugging) and we can use it to debug Python code running on Docker.

First, we have to configure our app container to expose the port that we want pudb to listen to. For this example, we’ll expose it in the port 6900.

If using Docker Compose, we can add a `docker-compose.override.yml` file in the project’s folder with the contents below:

```yaml
version: '2'
services:
  my-service-container-name:
    ports:
      # expose any that ports you have to expose
      - 8081
      # pudb will be exposed via 6900
      - "6900:6900"
```

Once that’s in place, we can drop the lines below wherever we want to inspect the service execution:

```python
from pudb.remote import set_trace
# replace (120, 60) with your terminal dimensions
set_trace(term_size=(120, 60), host='0.0.0.0', port=6900)
```

Now, once our program stopped its execution in that line, we can just connect to pudb via telnet:

```bash
$ telnet 0.0.0.0 6900
```

And *voilá*, pudb should open as if running locally. 🙂

# Wrapping up
**[Pudb](https://github.com/inducer/pudb)** rocks and I wish that more people would use and promote it. While **pdb** and other debuggers work pretty well, I think that they are not very intuitive and can be a rough experience for beginners.

Go ahead, give **pudb** a try and let your friends/colleagues know about it. I am very grateful to my friend that introduced me to it.


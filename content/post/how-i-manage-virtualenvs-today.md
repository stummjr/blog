---
title: "How I manage Virtualenvs with Pyenv"
author: "Valdir Stumm Jr"
tags: ["coding", "python"]
date: 2021-02-14T19:50:13-03:00
author: Valdir Stumm Jr
draft: false
---

I have always been a happy [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) user, but I abandoned it last year to use [pyenv](https://github.com/pyenv/pyenv) and [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv). I don't really remember why, as virtualenvwrapper is awesome.

The problem is that lately I haven't been creating and managing a lot of virtualenvs, so I often find myself having to search through pyenv docs to do basic stuff when needed. That's why I
wrote down the usual steps that I follow so that next time I can remember (or find) more easily.

**Note:** this is not a comprehensive guide or tutorial on virtualenvs and pyenv. It's just a collection of notes that I had on my note taking app that I thought that could be useful to someone else.

## Installing pyenv and pyenv-virtualenv
Here's how I install it on my Mac + Zsh:

```
$ brew install pyenv pyenv-virtualenv
$ echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
$ echo 'eval "$(pyenv init -)"' >> ~/.zshrc
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
```

There are other ways to install it, as you can see [here](https://github.com/pyenv/pyenv#installation).


## Updating the list of Python versions
Every now and then a new Python version comes out and you'll want to be able to install it.
To do that, you'll have to update the list of Python versions that are available on your
local machine.

If pyenv was installed via homebrew as I did above, you can just run:

```
$ brew upgrade pyenv
```

If you manually installed pyenv by cloning the project repository, then you can run
this command to update your local copy of the repo:

```
$ cd `pyenv root` && git pull
```


## Installing a new Python version
First, list the versions available for installation:
```
$ pyenv install  --list
```

If the version that you're looking for is not listed, try the commands listed in the previous topic.

Once you found the target version on the list (let's say it's `3.8.3`), you can install it via:

```
$ pyenv install 3.8.3
```

## Creating a virtualenv with a specific version

Let's say that you want to create a new virtualenv using the version you just installed (`3.8.3`). You can do so via:

```
$ pyenv virtualenv 3.8.3 my-venv-3.8.3
```

## Listing virtualenvs

```
$ pyenv virtualenvs
```

## Activating a virtualenv

```
$ pyenv activate my-venv-3.8.3
```

## Deactivating a virtualenv

```
$ pyenv deactivate
```

## Activating a virtualenv by default on a project

Let's say that you have a project on a folder in your filesystem and you always want to activate
a given virtualenv when entering that folder. To do that, all you have to do is to place a
`.python-version` file in that folder with the name of your virtualenv:

```
$ echo "my-env-3.8.3" > .python-version
```

That's it, now whenever you enter that folder, the virtualenv will be activated. This also
works for Python versions that you have installed via `pyenv`, not just virualenvs.

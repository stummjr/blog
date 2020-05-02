---
title: "Building a custom Flake8 plugin"
date: 2018-12-31T15:42:35-03:00
tags: [python, linter, flake8, ast]
draft: false
---

Linters are everywhere. Be it in a fancy IDE, a CI pipeline or in the command line, linters help us to spot potential issues in our codebases. My favorite linter is [flake8](http://flake8.pycqa.org/en/latest/) and I use it in my VSCode setup, in my [git pre-commit hooks](http://flake8.pycqa.org/en/latest/user/using-hooks.html#built-in-hook-integration) and CI pipelines.


But the thing is that flake8 doesn’t catch all the stuff I wanted it to catch. For example, I’d like my linter to catch the usage of the `map` and `filter` functions.

So I was wondering how can I make my linter complain about it every time someone else or I write such a thing? The answer is: **let’s write a flake8 plugin!**

To do so, we’re gonna use two tools:

- **ast:** the stdlib module to manipulate Python Abstract Syntax Trees;
- **flake8:** one of the (many) Python linters;

# The anatomy of a flake8 plugin

For a simple plugin, all we need is to create two files in a new folder (which I called `flake8-picky/`):

- `setup.py`: to make it installable and distributable;
- `picky_checker.py`: the module for the code checker itself.

Let’s start with the boring stuff (**setup.py**) so that we are free to have fun hacking our plugin later. Here it is:

```python
import setuptools
 
setuptools.setup(
   name='flake8-picky',
   license='MIT',
   version='0.0.1',
   description='A plugin to pick on map and filter usage :)',
   author='Your name here',
   author_email='you@yourdomain.com',
   url='http://github.com/yourname/your-repo',
   py_modules=['flake8_picky'],
   entry_points={
       'flake8.extension': [
           'PCK0 = picky_checker:PickyChecker',
       ],
   },
   install_requires=['flake8'],
   classifiers=[
       'Topic :: Software Development :: Quality Assurance',
   ],
)
```

Apart from the usual `setup.py` stuff, there’s a section we need to pay attention:

```python
entry_points={
    'flake8.extension': [
        'PCK0 = picky_checker:PickyChecker',
    ],
}
```

We’ve listed a single entry point for our flake8 plugin, which is the `picky_checker.PickyChecker` class (we’ll get there soon). As you can see, we’ve listed it under the `'flake8.extension'` entry point type, because this is what we need for a plugin that will add code verifications to Flake8. You can check for more options in the [official docs](http://flake8.pycqa.org/en/latest/plugin-development/registering-plugins.html?highlight=flake8.extension).

Another thing to notice here is the string we added to the list of entry points: `'PCK0 = picky_checker:PickyChecker'`. `PCK0` is a code prefix for the kind of issues we are going to report (they must all start with such a substring).

Now let’s focus on the `picky_checker.py` file which will contain:

- a class to parse and check the code to be linted;
- the entrypoint class for our plugin.

We’ll start with the former.


# Building our checker with ast

It doesn’t surprise me that an awesome language like Python has a module in the stdlib that allows us to easily parse Python code: the ast module.

The ast module provides the [ast.NodeVisitor](https://docs.python.org/3/library/ast.html#ast.NodeVisitor) base class, which basically walks through the [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) calling visitor functions for every node it finds.

For example, let’s say we want to find all the function definitions in a Python snippet and print their names. Here’s how we’d do it using ast:

```python
>>> import ast
>>> class FunctionFinder(ast.NodeVisitor):
       def visit_FunctionDef(self, node):
           print('Found: {}'.format(node.name))
 
>>> sample = '''
   def myfunc():
     pass
   def anotherfunc(x, y):
     return x * y
 
   x = myfunc() + 1
   '''
>>> parsed = ast.parse(sample)
>>> finder = FunctionFinder()
>>> finder.visit(parsed)
Found: myfunc
Found: anotherfunc
```

Easy like that. So, if we want our plugin to focus on a specific kind of AST node, all we have to do is to implement the `visit_*()` method and add the checks inside. Check out the full list of node types here: https://greentreesnakes.readthedocs.io/en/latest/nodes.html

## The parser
Getting back to our flake8 plugin, the issue we want to catch is the `map` and `filter` usage. To check for that, all we have to write is a parser like this:

```python
import ast
 
 
class ForbiddenFunctionsFinder(ast.NodeVisitor):
    forbidden = ['map', 'filter']
 
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.issues = []
 
    def visit_Call(self, node):
        if not isinstance(node.func, ast.Name):
            return
 
        if node.func.id in self.forbidden:
            msg = "PCK01 Please don't use {}()".format(node.func.id)
            self.issues.append((node.lineno, node.col_offset, msg))
```

In this case, our checker will visit each and every `ast.Call` node in the AST and check if its name is not one of the forbidden functions.

As you can see, we add some information into the `issues` list whenever we find a call. One important thing to remember here is that the linter error message should start with an error code that matches the prefix defined in `setup.py` for our linter (`PCK0` in our case).

I think we’ve got enough information about how `ast.NodeVisitor` works in order to build our plugin, so let’s move to the entry point class.


## The entry point
Let’s create the `picky_checker.py` file and add the entry point code on it:

```python
class PickyChecker(object):
    options = None
    name = 'picky-checker'
    version = '0.1'
 
    def __init__(self, tree, filename):
        self.tree = tree
        self.filename = filename
 
    def run(self):
        parser = ForbiddenFunctionsFinder()
        parser.visit(self.tree)
 
        for lineno, column, msg in parser.issues:
            yield (lineno, column, msg, PickyChecker)
```

Most of this is boilerplate, but let’s focus on the `run()` method. This method is the one called when Flake8 runs the verifications. There, we first instantiate our `ForbiddenFunctionsFinder` class, which will be basically an `ast.NodeVisitor` doing the verifications. Once we have the object, we call the `.visit()` method so that our node visitor traverses the AST.

After that’s done, we iterate over the issues found by `ForbiddenFunctionFinder` generating tuples with the issues in the order expected by Flake8: line number, column number, the linter message and the class that found the issues.

# Gluing it all
We’ll end up with the following files in our plugin folder:

```
├── picky_checker.py
└── setup.py
```

The `picky_checker.py` file should contain both the `ForbiddenFunctionsFinder` and `PickyChecker` classes.

# Installing our linter

Now that we have our linter code in our `flake8_picky` folder, let’s install our plugin and run flake8 over some sample files. You can install it by running:

```bash
$ pip install .
```

Then you can check if the plugin got installed by running:

```bash
$ flake8 --version
3.5.0 (mccabe: 0.6.1, pycodestyle: 2.3.1, pyflakes: 1.6.0, picky-checker: 0.1)
```

Finally, create some sample files and run flake8 against them:

```bash
$ flake8 samples/01.py
samples/01.py:4:5: PCK01 Please don't use map()
samples/01.py:7:5: PCK01 Please don't use filter()
```

# Wrapping up

That’s it. All we need to build a flake8 plugin is:

- a `setup.py` file to make it installable;
- an entrypoint class that will run your code checker;
- the code checker itself, which can be a `NodeVisitor` subclass.

Here you can find a repo with the linter developed here: https://github.com/stummjr/flake8-picky/

If you’re angry at me because you love `map` and `filter`, please forgive me as I had to come up with an example. 🙂

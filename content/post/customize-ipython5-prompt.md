---
title: "How to customize your IPython 5+ prompt"
author: "Valdir Stumm Jr"
tags: ["python", "ipython", "shell"]
date: 2018-09-02T15:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

**[IPython](https://ipython.org/) is wonderful and I ❤️ it.** I can’t see myself using the default Python shell in a daily basis. However, its default prompt kind of annoys me:

![](/img/posts/ipython1.png)

**Some of the things that I dislike:**

- the banner displayed when we start it;
- the `In[x]` and `Out[x]` displayed for inputs and outputs;
- the newline in between commands;
- and last, but far from least, the uber-annoying *“do you really want to exit?”* message.

As you can see, it doesn’t take much to get on my nerves. 😆

The bright side is that it’s easy to change that and have a more pleasant experience with IPython. This is my ideal shell, more compact and less bureaucratic:

![](/img/posts/ipython2.png)

If you like it, follow me through the next steps to make your IPython shell look and behave like that.

# Customizing the prompt
**First** you have to create a default profile for your shell with this command:

```bash
$ ipython profile create
```

As a result, a `.ipython` folder will be created in your home folder, with the following contents:

```
.ipython
├── extensions
├── nbextensions
└── profile_default
    ├── ipython_config.py
    ├── log
    ├── pid
    ├── security
    └── startup
        └── README
```

**Next**, create  `.ipython/custom_prompt.py` file with the following content:

```python
from IPython.terminal.prompts import Prompts, Token
 
class CustomPrompt(Prompts):
 
    def in_prompt_tokens(self, cli=None):
        return [(Token.Prompt, '>>> '), ]
 
    def out_prompt_tokens(self, cli=None):
        return [(Token.Prompt, ''), ]
 
    def continuation_prompt_tokens(self, cli=None, width=None):
        return [(Token.Prompt, ''), ]
```

**And last**, you have to tell IPython to use this new class as your prompt and in addition to custom settings.

You can do so by adding this code to `.ipython/profile_default/ipython_config.py`:

```python
from custom_prompt import CustomPrompt
 
 
c = get_config()
 
c.TerminalInteractiveShell.prompts_class = CustomPrompt
c.TerminalInteractiveShell.separate_in = ''
c.TerminalInteractiveShell.confirm_exit = False
c.TerminalIPythonApp.display_banner = False
```

**That’s it**, now you have a prompt like the one I’ve shown earlier. I hope it improves your experience with IPython as it did for me.

If you want to learn how to do further customizations, check [the official documentation](https://ipython.readthedocs.io/en/stable/config/details.html#custom-prompts).

Ah, did I mention that **I love IPython**? Huge kudos and thanks for the team behind it! 👏

---
title: "Black formats my code, and it should format yours too"
author: "Valdir Stumm Jr"
tags: ["python", "tools", "libraries"]
date: 2020-05-23T10:19:13-03:00
author: Valdir Stumm Jr
draft: false
---

I’ve always been a bit skeptical about code formatters. I don’t know, I always felt like
they would curb my freedom to format the code in my way. *Because, you know, no one formats code
better than me. Are you telling me that a tool does? Get out of here!*

Joking aside, I got to know [`black`](https://github.com/psf/black) about 2 years ago.
Everyone was talking about it. A bunch of people adopted it.
Massive codebases were being reformatted daily. It was the new kid on the block.

My initial reaction? Contempt. I didn't want to use a code formatter that did not allow me to customize
it to format the code the way I like. *Double quotes? Get the hell out of here!*

> Black is opinionated and so was I.

Silly me. **It should have never been about me or my own taste**, but the opposite. Black's goal is
to make Python codebases all around look at least similar in their format.


# What changed my mind?
Fast forward one year and I see myself posting a lot of change requests in PRs asking people
to format their code to match **my personal preferences**. That attitude can delay PRs and trigger
long and frustrating discussions.

Perhaps having coding guidelines outlining the rules on code style could have helped. It would certainly have
helped onboarding new team members. While we all have PEP-8 as a common idiom, there are many issues that go
beyond what's defined there and that's why having a written reference is always a good idea.

I do not think that coding guidelines is the ultimate solution though. People will challenge what's defined
there. Discussions will still take place. Your guidelines will have several gaps that will leave margins for
pure interpretation.

That's exactly why these days I think that not being highly customizable is black's greatest strength.
Once you adopt it, your "code style czar" badge will be instantly dropped. And what a relief!


## How do I feel about it now?
I love `black`. Code reviews these days have less bike-shedding and more meaningful contributions. They
focus on what really matters, basically. Don't get me wrong, I do think that style matters, but we now have
`black` as our (not so) benevolent dictator in any discussion regarding that. The code doesn't necessarily
look exactly how I would like it to look. But at least there is consensus now and `black` is always ready
to take the fall. No hard feelings at all.


# A word of advice, if I may
I work on a small team, in a relatively new codebase well covered with tests. We rarely have
more than 20 pull requests open simultaneosuly. That all made it easier to start using black.

Once you decide to adopt it, you'll want to reformat your whole codebase using it. That means that
most of your open PRs will have some sort of conflict, and that can be a pain if you have tons of them.
The PR reformatting your code will likely be humongous, and careful reviews will be required to make sure
nothing gets messed up. If your codebase is not well tested, this can become even more daunting.
A solution to this may be to apply `black` incrementally in your codebase. Check out this Github Action
to help you with that: [Gradual Black Formatter](https://github.com/marketplace/actions/gradual-black-formatter).

Finally, your revision history will now have a huge "Reformat codebase" commit under your name. If you have the
habit of digging into your project's revisions, I am sure you would not like that much. The good news is that
`git blame` allows you to ignore specific revisions so that they don't show up when you are scavenging
commits. You can do that via the
[`--ignore-rev`](https://git-scm.com/docs/git-blame#Documentation/git-blame.txt---ignore-revltrevgt) and
[`--ignore-revs-file`](https://git-scm.com/docs/git-blame#Documentation/git-blame.txt---ignore-revs-fileltfilegt)
options.

This section is not meant to discourage you, as adopting black is worth the potential trouble.
I just want you to know that you may face some roadbumps to get there.


# A suggested setup
Once you managed to apply `black` to your whole codebase, you have to make sure that any new changes
will be `black`-compliant. The easiest, but not so effective, way to do that is by kindly asking everyone
to run `black` before any commit. Don't get me wrong, it's not that I don't trust people to run it. The
thing is that we're all humans and we'll just forget it.

My team enforces `black` via git commit hooks. To do that, we use the excellent
[`pre-commit`](https://pre-commit.com/) package and ask all the team members to run `pre-commit install`
in their local setup. Once everyone does that, no one will be allowed to even commit their changes locally
in case there are violations.

This is somewhat effective. But, as I said before, we're all humans and humans forget stuff. I did forget it
once (*or maybe twice ...*) when setting up the development environment in new machines. Thankfully, the
project has a CI setup that fails the PR build in case `black` detects violations.

So this is what I suggest you to do:

1. Use `pre-commit` to enforce `black` in local commits.
2. Make sure your "Contribution Guidelines" doc provides the installation instructions.
3. Setup a check on your CI to fail the build in case `black` detects violations.

This can all be easily achieved with pre-commit and github actions. I've created a very simple
project to demonstrate that setup: https://github.com/stummjr/black_setup_project


# Wrapping up
These days, I am a huge fan of `black`. Of course, there are still some lingering pet-peeves. But that's just because
black is as opinionated as me. Black ain't gonna change, but I can. :)

Setting up black worked pretty well on my team, and may be worth a shot on yours as well.

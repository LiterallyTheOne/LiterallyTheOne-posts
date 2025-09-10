---
date: '2025-07-24T21:13:00+03:30'
draft: false
title: 'Macro in Vim'
description: "A post about Macro in Vim"
tags: ["vim"]
image: "macro-in-vim.webp"
---

# Macro in Vim

## Introduction

Today, I wanted to change some parts of my code that had a repetitive sequence
of actions.
So, I thought it was a good idea to use `macro` in `vim`.
`Macro` gives you the ability to record a sequence of actions and then
repeat them as many times as you want.

## My use-case

I had a code like below:

```python
report["train"]["loss"] = metrics["loss"]
report["train"]["accuracy"] = metrics["accuracy"]
report["train"]["precision"] = metrics["precision"]
report["train"]["recall"] = metrics["recall"]
report["train"]["f1"] = metrics["f1"]
```

I wanted to change it to the code below:

```python
report["train"]["loss"].append(metrics["loss"])
report["train"]["accuracy"].append(metrics["accuracy"])
report["train"]["precision"].append(metrics["precision"])
report["train"]["recall"].append(metrics["recall"])
report["train"]["f1"].append(metrics["f1"])
```

So what I did was, I started recording my macro by pressing these keys:

```sh
qa
```

* `q`: starts the recording
* `a`: the register that I write to

First thing that I am going to do is to go to the start of the line
using `^`.
Then, I find the first space using `f` followed by a space (` `).
After that, I press `v` and repeat the finding by pressing `;`.
Now, I have ` ` `=` ` ` selected.
Then, I press `s` to delete them and enter the insert mode.
Next, I type `.append(` and press `Esc` to enter the normal mode.
After that, I type `$` to go to the end of the line and type `)`.
Then, I press `j` to go to the next line.
And that is my macro.
Now, if I type: `:reg a`, I would have something like below:

```shel
Type Name Content
c  "a   ^f v;s.append(^[$a)^[j
```

* `c`: stands for change
* `"a`: means register is saved in `a`
* `^f v;s.append(^[$a)^[j`: is the sequence that we just did

To make the change, I only need to press `@a` to apply the changes
that I want.
Also, if I want, I can do `4@a` for the 4 remaining lines that I have.

## Final thoughts

`Macro` in `vim` is a powerful tool to do a sequence of events repetitively.
I really enjoy using `vim` and I am planning to learn it better.
But not going to lie, I think the AI assistants are going to replace
these parts of `vim` very soon.
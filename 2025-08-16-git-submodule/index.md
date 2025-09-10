---
date: '2025-08-16T17:36:00+03:30'
draft: false
title: "Git submodule and sparse-checkout"
description: "A post about how I used git submodule and sparse-checkout to add tutorials"
tags: ["Git", "GitHub"]
image: "submodule-git.webp"
---

# Git Submodule and sparse-checkout

## Introduction

So, I was trying to find the best way to put tutorials into my site.
I thought of every tutorial as an individual that has **docs**, **codes**, and e.t.c.
After some research, I found out that the best way to do that is to create a repository
for each tutorial that I have.
Then put them in my site using `git submodule`.
And if I only want the **docs** to be included, I can use `git sparse-checkout`.

## Add a repository as a git submodule

I created a new repository on **GitHub**.
My tutorial section is in this path: `site/content/tutorials`.
So, I wanted to use `git submodule` to include it on this specific path.
To do that, I have executed the code below:

```shell
git submodule add https://github.com/LiterallyTheOne/Pytorch_Tutorial site/content/tutorials/pytorch
```

This code has created a file called `.gitmodules` with the content below:

```text
[submodule "site/content/tutorials/pytorch"]
	path = site/content/tutorials/pytorch
	url = https://github.com/LiterallyTheOne/Pytorch_Tutorial
```

And now, I have access to my other repository.

## Use git sparse-checkout to only check docs

I wanted to only have the **docs** directory of my repository.
To do that, I executed the code below:

```shell
cd content/tutorials/pytorch
git sparse-checkout init --cone
git sparse-checkout set docs
```

First, I went to the directory of my tutorial.
Then, I initialized the `sparse-checkout`.
And finally, I set the `sparse-checkout` to **docs**.
Now, the other directories are not being included in my site repository.

## Final thoughts

`Git` is a powerful tool with so many great features.
`Git submodule` helps you to include other repositories in your repository.
It is way more convenient than copying all the code.
`Git sparse-checkout` helps you to check out certain files.
I used both of them to include my tutorials on my site.

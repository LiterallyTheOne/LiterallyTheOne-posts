---
date: '2025-07-15T22:24:09+03:30'
draft: false
title: 'inoremap in vim'
tags: ["vim", "PyCharm"]
image: "inoremap.webp"
---

# inoremap in vim

`inoremap` stands for `Insert-mode Non-recursive Map`.
It is a really helpful command in `vim` which helps
you to `map` things in `Insert-mode`.
For example, if you want to insert `Hello world`, using `\a`
in `Insert mode`,
You can have a code like below:

```shell
inoremap <Leader>a Hello world
```

In the example above:

* `<leader>`: your leader key, usually: `\`

Or if you want to exit the `Insert-mode` using `jj`,
you can have something like this:

```shell
inoremap jj <Esc>
```

For convenience, you can put these codes in your `vim` config.
One of the `vim` config files that you can edit is: `~/.vimrc`.

## How I use it

I really like to have my headers in markdown centered, and have
their own colors.
In order to do that, I have set some `inoremap`s to help me:

```shell
inoremap <Leader>h1 # <div style="text-align: center; color: cyan"></div><Esc>F<i
inoremap <Leader>h2 ## <div style="text-align: center; color: lime"></div><Esc>F<i
inoremap <Leader>h3 ### <div style="text-align: center; color: lightsalmon"></div><Esc>F<i
```

In the codes above:

* `<Leader>`: `\`
* `<Esc>`: exit the insert mode
* `F<`: backward finds the `<`
* `i`: enters insert mode again

So, right now, anytime that I want to insert a header, I can make
sure that I am in insert mode and type on of the following:

* `\h1`: header 1
* `\h2`: header 2
* `\h3`: header 3

## Config in Pycharm

For normal `vim`, we could have put our codes in `~/.vimrc`.
For `vim plugin` in `Pycharm`, the only thing that we should do
is to put it in `~/.ideavimrc`.
---
date: '2025-07-16T15:32:00+03:30'
draft: false
title: 'How I use TailwindCSS Typography'
description: "A post about how I use the TailWindCSS Typography"
tags: ["CSS", "TailwindCSS", "TailwindCSS-typography"]
image: "tailwindcss-typography.webp"
---

# How I use TailwindCSS typography

I have recently switched from `bootstrap` to `TailwindCSS` and I am so
happy about it.
It is super simple and modifiable.
I used to use `django` for my site and to render `markdown` files to
`html`, I was using `markdown` package in `python`.
Then, with the help of `regular expresseion`, I was modifying the rendered
result.

Now, I use `hugo` for my static site.
After `hugo` renders my `markdown` files to `html`, I use
[`TailwindCSS Typography`](https://github.com/tailwindlabs/tailwindcss-typography)
to make the result look good.

## Install TailwindCSS Typography

To install `TailwindCSS Typography`, at first, you need to install
it using `npm`:

```shell
npm install -D @tailwindcss/typography
```

Then, you should just add it to your `main.css` like below:

```css
@plugin "@tailwindcss/typography";
```

And you are good to go.

## My use case

I use the `TailwindCSS Typography` like below anywhere that
I have a rendered `markdown`:

```html

<article
        class="prose dark:prose-invert lg:prose-xl prose-headings:text-center prose-h1:text-cyan-500 prose-h2:text-lime-500 prose-h3:text-orange-500 prose-inline-code:text-purple-500 prose-inline-code:rounded prose-inline-code:bg-gray-200 prose-inline-code:before:content-none prose-inline-code:after:content-none">
    {{ .Content }}
</article>
```

* `prose`: Tells the `TailwindCSS` to render it
* `dark:prose-invert`: make it compatible for dark theme
* `lg:prose-xl`: make it compatible for large displays

---

* `prose-headings:text-center`: centers all the headings
* `prose-h1:text-cyan-500`: colors the `<h1>` with cyan
* `prose-h2:text-lime-500`: colors the `<h2>` with lime
* `prose-h3:text-orange-500`: colors the `<h2>` with lime

---

* `prose-inline-code:text-purple-500`: Makes the inline-code purple
* `prose-inline-code:rounded`: adds a rounded box around the inline-code
* `prose-inline-code:bg-gray-200`: makes the background of the inline-code gray
* `prose-inline-code:before:content-none`: removes "`" from the start of the inline-code
* `prose-inline-code:after:content-none`: removes "`" from the end of the inline-code

### Important note

In the current version that I'm using
( [v0.5.16](https://github.com/tailwindlabs/tailwindcss-typography/releases/tag/v0.5.16) )
`prose-inline-code` is not implemented.

So with the suggestion of this comment in `GitHub issues`:
[link](https://github.com/tailwindlabs/tailwindcss-typography/issues/329#issuecomment-2628814973)
I added the code below to my `main.css`:

```css
@custom-variant prose-inline-code (&.prose :where(:not(pre)>code):not(:where([class~="not-prose"] *)));
```




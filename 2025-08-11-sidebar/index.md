---
date: '2025-08-11T16:47:00+03:30'
draft: false
title: "Add sidebar and shortcodes to tutorial's section"
description: "Add a post about adding a sidebar and shortcodes to tutorial's section"
tags: ["hugo", "TailwindCSS"]
image: "sidebar-and-shortcodes.webp"
---

# Add sidebar to tutorial's section

## Introduction

Today, I decided to make a better template for my tutorial's section.
I really loved the way
[TailwindCSS docs](https://tailwindcss.com/docs/installation/using-vite) and
[Jupyterbook](https://jupyterbook.org/en/stable/intro.html) are handling
their documentation.
Their method is pretty clean, and you can navigate through it pretty easily
and find whatever you are after.
So, I tried to make a style like theirs.

## Create sidebar

To create a sidebar, I have made a `partial template`,
named `sidebar-tutorial.html` on `hugo` like below:

```html
{{ $current := .RelPermalink }}

{{ range where site.Sections "Section" "tutorials" }}

<div class="flex flex-col dark:text-white rounded border-4 border-blue-200 dark:border-blue-800">

    <!--@formatter:off-->
    {{ range .Sections }}
    <div class="flex">
        <a href="{{ .RelPermalink }}" class="hover:underline font-bold
        {{ if eq .RelPermalink $current }}text-blue-500 dark:text-blue-300 underline{{ else }}text-gray-800 dark:text-gray-500{{ end }}">
            {{ .Title }}
        </a>
    </div>
    {{ range .Pages }}
    <div class="ml-4">
        <a href="{{ .RelPermalink }}" class="hover:underline
        {{ if eq .RelPermalink $current }}text-blue-500 dark:text-blue-300 underline font-semibold{{ end }}">
            {{ .Title }}
        </a>
    </div>
    {{ end }}
    {{ end }}
    <!--@formatter:on-->


</div>
{{ end }}

```

As you can see, at first, I get the `link` to the page that is called
`sidebar-tutorial.html` and store it in `$current`.
After that, I iterate through `sections` in `tutorials`.
It has only one element, and it is named `tutorials`.
Not going to lie, I haven't found a better solution yet, but I'm going
to fix it in the near future.
Then, I create a `div` to style my `sidebar`.
Next, I have a loop to go through all the `sections`
(subsections of `tutorials`, e.g., `Pytorch`).
After that, I iterate over all the pages of that section and show them
in the format that I want.
As it is shown, I have an `if` statement in each link (`<a>`) that I make.
These `if` statements are used for highlighting where we are right now,
using `$current`.
The reason that I put `<!--@formatter:off-->` and `<!--@formatter:on-->` is
that I'm using `Pycharm` and it doesn't support `go-template`.
Every time I formatted the code, it made my `go-template` much worse.

## Add table of contents

In `jupyterbook`, we could put a table of contents by
using `{tableofcontents}` in our markdown.
So, I thought it would be a good idea to implement that on my site as well.
To do that, I created a file named `table-of-content.html` in `layouts/shortcodes`
with the following content:

```html
<div class="flex flex-col dark:text-white rounded border-4 border-blue-200 dark:border-blue-800">

    <!--@formatter:off-->
    {{ if len page.Sections | ne 0 }}
    {{ range page.Sections }}
    <div class="flex">
        <a href="{{ .RelPermalink }}" class="hover:underline font-bold text-gray-800 dark:text-gray-500">
            {{ .Title }}
        </a>
    </div>
    {{ range .Pages }}
    <div class="ml-4">
        <a href="{{ .RelPermalink }}" class="hover:underline">
            {{ .Title }}
        </a>
    </div>
    {{ end }}
    {{ end }}
    {{ else }}
    {{ range page.Pages }}
    <div class="ml-4">
        <a href="{{ .RelPermalink }}" class="hover:underline">
            {{ .Title }}
        </a>
    </div>
    {{ end }}
    {{ end }}
    <!--@formatter:on-->


</div>
```

As you can see, the format is pretty similar to the `sidebar`.
One of the things that is different from the `sidebar` is that
if there were no sections, I would iterate through the pages.
Also, there is no highlighting.
Now, every time that I want to have a `table of contents` in my `markdown`,
I can simply use the code below:

```html
{{ < table-of-content > }}
```

This simple line tells `hugo` to use the `html` file that I created in
the `shortcodes`.
(There are extra spaces in the code above to prevent `hugo` from parsing it)

## Final Thoughts

Having a good structure for tutorials is extremely helpful.
My preferred way is the way `jupyterbook` does that.
So, I did my best to implement the structure in a way that I want with
the influence of `jupyterbook` and `TailwindCSS docs`.
For the next step, I'm trying to put a `tutorial` for `PyTorch`.
I'm going to write that tutorial with examples and explain it
in a way that I like any tutorial to be.




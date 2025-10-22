---
date: '2025-10-22T14:05:00+03:30'
draft: false
title: "Create slides with Marp"
description: "A post about creating slides using Marp"
tags: [ "Marp", "vs-code" ]
image: "marp-slides.webp"
---

# Create slides with Marp

## Introduction

To present my tutorials, I thought it would be a good idea to create slides.
**Marp** is one of the greatest tools that we can use to create slides from markdown files.
It has various themes and can be exported to different formats, **HTML**, **PDF**, **PowerPoint**, etc.
It was perfect for me, because I could create slides with markdown and export them to **HTML** to put them in my site.
Here is the link to the official site of **Marp**: <https://marp.app/>.
I highly recommend that you give it a try.

## Create slides

For the first step, we need to have **Marp** installed.
The fastest and easiest way is to use the official **VS-Code** extension.
After installation, we should create a **Markdown** file and modify its frontmatter.
For example, I configured the frontmatter like below:

```text
---
marp: true
theme: uncover
footer: By Ramin Zarebidoky (LiterallyTheOne)
header: Arduino Tutorial, intro
size: 16:9
---
```

We can separate slides using three hyphens like this `---`.
For the first slide, which is my title slide, I do not want to have a header and a footer.
Also, I want to write my name in cyan.
To do so, I use the code below:

```text
<style scoped>
p {
  color: cyan;
}
</style>

<!-- _header: "" -->
<!-- _footer: "" -->

```

For page numbers, which I want to start from the second slide, we can use the code below in the second slide:

```text
<!-- paginate: true -->
```

Here is an example of the **Markdown** file that I have created for an **Arduino Tutorial**:
<https://github.com/LiterallyTheOne/Arduino-Tutorial/blob/master/slides/1-gpio/1-gpio.md>

## Export to html

To export to **HTML**, we can use **VS-Code** or **Command line**.
The most straightforward way is to use **VS-Code**.
In the top right of the page, there is a button with the name of
**Show quick pick of Marp commands ...**.
If you press it, you can see **export slide deck**.
Then a window will pop up.
In this window, you can choose which format you want your slides to be exported to.
I use **HTML** because I have GIFs in my slides, and I want to put them on my website.
In **PowerPoint** and **PDF** formats, GIFs are not showing correctly.
After I export them, I change the image paths and put them in a folder in `/static` to be able to link them in each
tutorial.
Here is an example of the **HTML** output:
<https://literallytheone.github.io/slides/tutorials/arduino/docs/1-gpio/1-gpio.html>

## Final Thoughts

**Marp** is a fantastic tool to create slides.
I use it to present my tutorials in my classes.
It is flexible and has various themes.
I really recommend that you give it a shot.
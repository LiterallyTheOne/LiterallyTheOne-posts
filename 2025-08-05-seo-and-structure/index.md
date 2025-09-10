---
date: '2025-08-05T16:47:00+03:30'
draft: false
title: 'Improve SEO and structure of my site'
description: "A post about improving SEO and struction of my website"
tags: ["SEO", "Google-Search-Console", "Site"]
image: "seo-improvement.webp"
---

## Improve SEO and structure of my site

Yesterday, I was searching for my name, and none of the results were pointing to my website.
So, I wondered what might be the problem.
After some research, I found a website that gives you insight into the `SEO` related things of your website.
The website is called: [seositecheckup.com](https://seositecheckup.com/).
I strongly recommend that you give it a try.
Also, I had some problems with the way I was showing the content of each post.
The posts were showing pretty well on my screen, but on mobile,
they had some problems that needed to be taken care of.

## Improve SEO

After I gave my website to [seositecheckup.com](https://seositecheckup.com/), it gave me some important points
that I didn't know.
First, URLs shouldn't have `underscore` on them.
Actually, because I was programming `Python`, I was accustomed to the `underscore` for naming my directories
and files.
After finding out it was a major problem, I suddenly changed all of the `underscores` to `hyphens`.
Also, I had a page called `about_me` and I changed it to `about`.

Second, it told me to install `Google Analytics` on my website.
It is totally free and also helps me with analyzing how users interact with my site.
So, I went to `Google Analytics` site and created a new account.
Then, it gave me a `JavaScript` code that I should have put on every page.
With the help of `cookie`, `Google Analytics` analyzes users' data.
The next thing that I have to do is to put a `cookie` notice.
So, with the help of `cookieconsent` I managed to do that.
The `javascript` code is below:

```html

<script src="https://cdn.jsdelivr.net/npm/cookieconsent@3/build/cookieconsent.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/cookieconsent@3/build/cookieconsent.min.css"/>
<script>
    window.addEventListener("load", function(){
      window.cookieconsent.initialise({
        palette: {
          popup: { background: "#000" },
          button: { background: "#f1d600" }
        },
        theme: "classic",
        content: {
          message: "This website uses cookies to enhance the user experience.",
          dismiss: "Got it!",
          link: "Learn more",
          href: "https://policies.google.com/technologies/cookies"
        }
      });
    });
</script>
```

## Improve structure

To improve the structure, I added `breadcrumbs`.
I saw a really great `breadcrumbs` code in the official `hugo` website.
So, I downloaded it and changed it in a way that I liked.
Here is the link to the source code:
[code](https://github.com/gohugoio/hugo/blob/master/docs/layouts/_partials/layouts/breadcrumbs.html)

After that, I fixed the problem with the posts.
I had a good perspective when I was looking at it on my MacBook.
But on phones and smaller devices, it had some problems.
First, I saw that `code blocks` are overflowing, and that makes
my page look awful.
So, I added `overflow-auto` to where I was using `TailwindCSS Typography`
and the problem was solved.
The other problem was the table of contents and the postcard.
When I was looking at them on my MacBook,
They were showing at the right of the window as intended.
But in the mobile version, they would still stick to the right side
which was not suitable at all.
So at first, I changed the `grid` to `flex` to have more comfort.
Then, I used two different versions of them.
One when the screen is `not small` and the other when the screen is `small`, like below:

```html

<div class="sm:hidden">
    <!--    When screen is small    -->
</div>

<div class="hidden sm:block">
    <!--    When screen is not small    -->
</div>
```

The code above worked out great.

## Final Thoughts

Applying `SEO` techniques are super important.
Especially when you have a small and personal site.
You want when someone searches for you on `Google`, your site would be at least in the top-3.
Tools like
[seositecheckup.com](https://seositecheckup.com/)
are super helpful to understand what parts need more work.
Also, having `breadcrumbs` is super useful in both `SEO` and the visitors' point of view.

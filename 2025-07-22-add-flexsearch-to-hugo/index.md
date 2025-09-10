---
date: '2025-07-22T09:47:00+03:30'
draft: false
title: 'Flexsearch on Hugo'
description: "A post about adding flexsearch in Hugo"
tags: ["flexsearch", "hugo", "javascript"]
image: "add-flexsearch-to-hugo.webp"
---

# Flexsearch on Hugo

## Introduction

I wanted to put a search feature on my website, and I wanted to find
The best way to do that.
The first thing that came to my mind was `Algolia`.
I should upload my `json` file to `Algolia` and it would index
it.
Then, by using its api, I could put a search button on my site.
After that, I remembered that `Jupyter Book` that I used to work with
has an integrated search feature.
So, I decided to see how they implemented that.
After some research, I found they use a `javascript` package called `Lunr.js`.
But it seemed that there were better packages like `Flexsearch`.
It appears to be the fastest search library for browsers.
So, I decided to use `Flexsearch` for now.

## Setup up Hugo

At first, I should tell `hugo` to create a `json` output as well.
To do so, I used the code below in `hugo.toml`:

```toml
[outputs]
home = ["HTML", "JSON"]
```

Then I should tell `hugo`, how to create the `json` file.
To do that, I made a file at this address `layouts/_default/index.json`.
Then, I put this code into that file:

```json
{{- $types := site.Params.searchTypes | default (slice "posts") -}}
{{- $pages := where .Site.RegularPages "Type" "in" $types -}}
[
{{- range $i, $page := $pages -}}
{{- if $i }},{{ end }}
{
"title": {{ $page.Title | jsonify }},
"description": {{ $page.Params.description | default $page.Summary | plainify | jsonify }},
"content": {{ $page.Plain | jsonify }},
"url": {{ $page.RelPermalink | jsonify }}
}
{{- end -}}
]
```

The code above goes through all the posts and extract their `title`,
`description`, `content`, and `url`.

## Install Flexsearch

After the step above, it is time to install `Flexsearch`.
To do that, I used `npm`.

```shell
npm install flexsearch
```

Then I copied the `flexsearch.bundle.min.js` file from `node_modules/flexsearch/dist`
to `js/vendor/`.

Then, to use the bundle, I have used the code below:

```html
{{ $flex := resources.Get "js/vendor/flexsearch.bundle.min.js" }}
<script src="{{ $flex.RelPermalink }}" integrity="{{ $flex.Data.Integrity }}"></script>
```

## Create a search button and page

At first, I put the search and their results to my header file.
Then, I thought it might look better if I have a separate page for search.
So, I only added the search button to `header.html` like below (I got the
`svg` from the official `tailwind` site):

```html

<div class="flex items-center gap-3 ">
    <a href="/search">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
             class="-ml-0.5 size-5 fill-gray-600 dark:fill-gray-500">
            <path fill-rule="evenodd"
                  d="M9.965 11.026a5 5 0 1 1 1.06-1.06l2.755 2.754a.75.75 0 1 1-1.06 1.06l-2.755-2.754ZM10.5 7a3.5 3.5 0 1 1-7 0 3.5 3.5 0 0 1 7 0Z"
                  clip-rule="evenodd"></path>
        </svg>
    </a>
</div>
```

Now, it's time to create a search page.
To do that, I have created a folder in `layouts` called `search`.
Then, in `layouts/search`, I created a file called `list.html`.
This file would have all the necessary things for the search.
So, I filled it in like below:

```html
{{ define "main" }}

<div class="flex flex-col m-4 justify-center gap-4">

    <div class="flex flex-wrap rounded  items-center justify-center">
        <div class="flex rounded border-4 border-blue-200 dark:border-blue-800">
            <div class="m-3 prose dark:prose-invert lg:prose-xl prose-headings:text-center prose-h1:text-cyan-500 prose-h2:text-lime-500 prose-h3:text-orange-500 prose-inline-code:text-purple-500 prose-inline-code:rounded prose-inline-code:bg-gray-200 prose-inline-code:before:content-none prose-inline-code:after:content-none">
                {{ .Content }}
            </div>
        </div>
    </div>

    <div class="flex flex-wrap rounded  items-center justify-center">
        <div class="flex rounded border-4 border-blue-200 dark:border-blue-800">
            <div class="dark:text-white">
                <label>
                    <input type="text" id="searchBox" placeholder="Search...">
                </label>
            </div>
        </div>
    </div>

    <div id="searchResults"
         class="flex flex-col gap-4 rounded shadow-md dark:text-white bg-white dark:bg-gray-950">
    </div>
</div>

{{ $flex := resources.Get "js/vendor/flexsearch.bundle.min.js" }}
<script src="{{ $flex.RelPermalink }}" integrity="{{ $flex.Data.Integrity }}"></script>


{{ $s := resources.Get "js/search.js"}}
<script type="module" src="{{ $s.RelPermalink }}"></script>

{{ end }}
```

As you can see in the code above, I have a search input and a `div`
for search results.
Also, I have included `flexsearch` bundle with `search.js` that I'm going
to explain it later.
As it was shown, I also have `{{ .Content }}`, but there is no file to
be rendered.
So, let's create that file in `content/search/_index.md`.
I only put the code below to that file:

```markdown
+++
title = "Search"
date = 2025-07-20
+++

# Search

```

As shown, it only has a header that says `Search`.

Now our page is ready, and we should implement the searching procedure.

## Searching procedure

To control the searching procedure, I have created `search.js`.
At first, we should create a document.

```js
const index = new FlexSearch.Document({
  tokenize: "forward",
  document: {
    store: true,
    id: "url",
    index: ["title", "description", "content"],
  }
});
```

The document above uses `url` as id and indexes `title`, `description`,
and `content`.
With the `store` set to true, it also stores them to make it easy
to retrieve.

Then, we should get data from the `json` file that we created with `hugo`
like below:

```js
async function loadData() {
const res = await fetch("/index.json");
const data = await res.json();
for (const doc of data) {
  index.add(doc);
}
}
```

The code above uses an `async` function to get data from `index.json`.
Then we add that data to the `document` we created earlier.

Then we define a function to do the search:

```js
function search(query) {
  const results = index.search({ query: query,
  enrich: true,
  suggest: true,
  highlight: {template: "<b class='rounded shadow-md bg-purple-500'>$1</b>", boundary: 100},

      });
  const flat = [
    ...new Set(results.flatMap(group => group.result))
  ];
  return flat;
}
```

The code above searched the query in the indexed documents.
* `enrich: true` makes sure that the result has the same structure
  as when we saved it.
* `suggest: true` if there wasn't an exact match, it suggests the documents 
  that have some part of the query.
* `highlight: true` highlights the match keyword.

Then, to render the output of the search, I added the code below:

```js
function render(results) {
  const container = document.getElementById("searchResults");
  container.innerHTML = "";
  for (const res of results) {
    const d1 = document.createElement("div");
    d1.classList.add("rounded");
    d1.classList.add("border-2");
    const a = document.createElement("a");
    a.classList.add("bg-cyan-500");
    a.href = res.doc.url;
    a.innerHTML = res.doc.title || res.doc.url;  // allows HTML (like <mark>)
//    a.textContent = res.doc.title || res.doc.url; // fallback to URL if title is missing
    d1.appendChild(a);

    // Optionally show description
    const desc = document.createElement("p");
    desc.innerHTML = res.highlight;
    d1.appendChild(desc);

    container.appendChild(d1);
  }
}
```

The code above finds `searchResuls` that I added in `search/list.html`
and adds the results to it with the format that I want.
Right now, the format is a little bit basic.
I should work on it more in the future.

And for the last thing, we should add a handler that, anytime someone
types something in the search box, the results show up.
To do so, I have added the code below:

```js
document.getElementById("searchBox").addEventListener("input", async (e) => {
const query = e.target.value;
if (!index.length) await loadData();
if (query.length > 1) {
  const results = search(query);
  render(results);
}else{
  const container = document.getElementById("searchResults");
  container.innerHTML = "";
}
});
```

Now, everything is set and we are good to go.

## Final thoughts

`Flexsearch` is a really great option to implement search in my site.
It is free and doesn't need any online accounts.
I was reading that it might be troublesome if I have too many posts
but right now, it works pretty well.
I strongly recommend that you give it a try.
Here is the link to their project: [flexsearch](https://github.com/nextapps-de/flexsearch)
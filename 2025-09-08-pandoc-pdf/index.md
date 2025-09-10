---
date: '2025-09-08T15:36:00+03:30'
draft: false
title: "Use pandoc to generate pdf"
description: "A post about how I use pandoc to generate pdf"
tags: [ "Pandoc", "pdf", "latex", "hugo" ]
image: "pandoc-pdf.webp"
---

# Use pandoc to generate pdf

## introduction

It's been a while since I wanted to generate a high-quality PDF for my
posts and tutorials.
I wanted them to be clean and simple.
Also, I wanted to use my **markdowns** that I have already written for my
**hugo website**.

## HTML to PDF

First, I was thinking that it might be a good idea to convert the **html**
files that I have generated with **hugo** to PDF.
But there were too many challenges.
I have tried different packages like **weasyprint**, **playwright**.
I was seeing that so many people were recommending using **wkhtmltopdf**, but after some research, I found out that
it is not being maintained anymore, and it's archived.
After that, I tried **pandoc** to generate PDF from HTML that I have already structured using my custom CSS.
But it didn't go well.
First, **pandoc** uses **LaTeX** to generate PDF.
Unfortunately, **LaTeX** doesn't support **WebP** images by default.
Second, my code snippets weren't formatted pretty well.
Also, I had to generate specific HTML for each **pandoc** document.
It was a little bit overdue in my opinion.
So, I decided to convert **Markdown** to **PDF**.

## Markdown to PDF

At first, I tried to convert a simple **Markdown** file with a block of code into a **PDF**.
It can be achieved as below:

```shell
pandoc input.md -o output.pdf --pdf-engine=xelatex
```

The code above converts the markdown file with the name of `input.md` to `output.pdf`.
I used `xelatex` for my `pdf engine`.
It wasn't that hard, but the configuration was a little bit tricky for me.
Because if the code was too long, it would have exceeded the width of the **PDF**.
To solve that, I found a solution.
I had to create a file called `header.tex` with this content:

```text
\usepackage{fvextra}
\fvset{breaklines, breakanywhere}
```

It tells the default code styler of **pandoc** to break the lines if it was needed.
There were other code stylers like **listings** and **minted** that I tried,
but this way sounded easier.
So, now I should tell **pandoc** to use my header file.

```shell
pandoc input.md -o output.pdf --pdf-engine=xelatex --include-in-header=header.tex
```

There are also highlighting styles that you can see the list of them using `pandoc --list-highlight-styles`.
I checked all the available ones and decided to use `tango`.
To add the highlight styler, I wrote the code below:

```shell
pandoc \
 input.md \
 -o output.pdf \
 --pdf-engine=xelatex \
 --include-in-header=header.tex \
 --highlight-style=tango
```

## Change frontmatter from toml to yaml

After converting one of my posts to **PDF**, I found that I should change the frontmatter from `toml` to `yaml`.
To do so, I wrote the code below:

```python
from pathlib import Path


def main():
    p = Path("site/content")
    for path in p.glob("**/*.md"):
        print(path)

        with open(path) as f:
            content = f.read()
            lines = content.split("\n")
            for i in range(15):
                lines[i] = lines[i].replace("+++", "---")
                lines[i] = lines[i].replace(" = ", ": ")

            result = "\n".join(lines)
        with open(path, "w") as f:
            f.write(result)


if __name__ == '__main__':
    main()
```

The code above finds all the `md` files.
Then it goes through `15` first lines and changes the frontmatter from `toml` to `yaml`.
It is not optimal, but it does the job that is intended to do.

## Final script

So, to automate this conversion procedure for all the files, I wrote the bash script below:

```shell
#!/bin/bash


current_dir="$(realpath .)"

dir_path=$(dirname "${current_dir}")/site/content

header_path="$current_dir"/header.tex
date_format_path="$current_dir"/date-format.lua

find "$dir_path" -name "index.md" -print0 | while IFS= read -r -d $'\0' file; do

    should_delete=("pandoc.md")

    parent_file=$(dirname "$file")

    destination_path=$(echo "$parent_file" | sed "s:/site/content/:/site/public/pdf/:g" )
    if [ ! -d "$destination_path" ]; then
      mkdir -p "$destination_path"
    fi

    echo "$parent_file"
    cd "$parent_file"

    shopt -s nullglob

    for img in *.webp; do
        magick "$img" "${img%.webp}.png"
        should_delete+=("${img%.webp}.png")
    done

    for img in *.gif; do
        magick "${img}[0]" "${img%.gif}.png"
        should_delete+=("${img%.gif}.png")
    done

    shopt -u nullglob

    sed "s:\.webp:\.png:g" index.md > pandoc.md
    sed -i "" "s:\.gif:\.png:g" pandoc.md

    pandoc \
        pandoc.md \
        -o "$destination_path"/output.pdf \
        --pdf-engine=xelatex \
        --metadata author="Ramin Zarebidoky (LiterallyTheOne)" \
        --include-in-header="$header_path" \
        --highlight-style=tango \
        --lua-filter "$date_format_path"

    for x in "${should_delete[@]}"; do
      rm "$x"
    done


done
```

As you can see, in the code above, at first, I find all the `index.md` files.
Then, I convert **WebP** and **GIF** files to **PNG**.
After that, I changed the references to them and stored the result in `pandoc.md`.
For the next step, I used pandoc to convert my **Markdown** file to **PDF**.
Finally, I deleted the files that weren't needed.
Also, I used a specific date format to make the date look better.
Here is the content of `date-format.lua`:

```text
function Meta(meta)
  if meta.date then
    local format = "(%d+)-(%d+)-(%d+)" -- Assuming date is in YYYY-MM-DD format in frontmatter
    local y, m, d = pandoc.utils.stringify(meta.date):match(format)
    if y and m and d then
      local date = os.time({ year = y, month = m, day = d })
      -- Change the format string below to your desired output format
      local date_string = os.date("%d %b %Y", date) -- Example: 02 Apr 2018
      meta.date = pandoc.Str(date_string)
    end
  end
  return meta
end
```

## Final Thoughts

**pandoc** is a really great tool to convert **Markdown** to **PDF**.
In my opinion, the best approach to have downloadable **PDF** files is to pre-generate them and put a link
to them on each page.
For the next step, I'm going to add my script to `CI/CD`.
Also, I want to create a **PDF** for all the tutorial files to look like a book. 



---
date: '2025-09-09T16:27:00+03:30'
draft: false
title: "Generating PDF using Pandoc in GitHub actions"
description: "A post about how I used GitHub actions to generate PDF with Pandoc"
tags: ["Pandoc", "pdf", "latex", "hugo", "GitHub", "github-actions", "docker"]
image: "pdf-github-actions.webp"
---

# Generating PDF using pandoc in GitHub actions

## introduction

Today, I wanted to fix the bug with changing the `toml` to `yaml` script and
try to use **GitHub actions** to generate PDFs on my website.

## Fixing the bug with changing `toml` to `yaml`

In the previous code that I had for changing `toml frontmatters` to `yaml frontmattes`
I was reading only the top 20 lines.
To be honest, it wasn't optimal at all, and I faced some bugs.
For example, in my code snippets, I had some ` = `, which was converted to `: `, and this is not acceptable at all.
So, at first, I reverted those comments that were affected by my code.
Then I changed the code as follows:

```python
from pathlib import Path


def main():
    p = Path("site/content")

    for path in p.glob("**/*.md"):
        print(path)

        state = 0

        with open(path) as f:
            content = f.read()
            lines = content.split("\n")
            for i in range(len(lines)):
                if state == 1:
                    lines[i] = lines[i].replace(" = ", ": ")

                if "+++" in lines[i]:
                    lines[i] = lines[i].replace("+++", "---")
                    state += 1

                if state == 2:
                    break

            result = "\n".join(lines)
        with open(path, "w") as f:
            f.write(result)


if __name__ == '__main__':
    main()
```

Now, I have a variable called state.
Every time that I see `+++`, `state` increments by 1.
So, it helps me to change only the parts surrounded by `+++`.
After that, I ran my code again and fixed the parts that were affected.

## Test GitHub workflow with act

Now that I have a way to generate PDFs, I thought it would be a good idea to write a `GitHub workflow`
for that.
I wanted to test my `GitHub workflows` on my local environment.
After some research, I found out that there is a package called `act` that we can use to test
our `GitHub workflows`.
So, I installed it using the code below:

```shell
brew install act
```

And to test all the workflows, I can use this code:

```shell
act
```

If I want to reuse the container that has been built and not install all the packages again,
I can use the code below:

```shell
act --reuse
```

Also, for a specific workflow, we can use `-W` option, like below:

```shell
act -W .github/workflows/workflow.yml --reuse
```

## Write a separate GitHub Workflow

At first, I wrote a separate file to generate PDFs.
My thought was that I would generate them for the public part of my site,
so those two can run separately.
I have had a workflow like this:

```yaml
name: Build PDFs with Pandoc

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Pandoc
        run: |
          sudo apt-get update
          sudo apt-get install -y \
          pandoc \
          texlive-latex-base \
          texlive-fonts-recommended \
          texlive-fonts-extra \
          texlive-latex-extra \
          texlive-xetex \
          librsvg2-bin
          sudo apt-get install -y imagemagick

      - name: Convert each md to pdf
        run: |
          bash convert_to_pdf_pandoc/convert_each_md_to_pdf.sh
```

At first, I installed all the necessary things for `pandoc` and `latex`.
After that, I run the script (`convert_each_md_to_pdf.sh`) that I modified to work with `GitHub actions`.
Here is the content of `convert_each_md_to_pdf.sh`:

```shell
#!/bin/bash


current_dir=$(dirname "$(realpath "$0")")

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
    cd "$parent_file" || exit

    shopt -s nullglob

    for img in *.webp; do
        convert "$img" "${img%.webp}.png"
        should_delete+=("${img%.webp}.png")
    done

    for img in *.gif; do
        convert "${img}[0]" "${img%.gif}.png"
        should_delete+=("${img%.gif}.png")
    done

    shopt -u nullglob

    sed "s:\.webp:\.png:g" index.md > pandoc.md
    sed -i "s:\.gif:\.png:g" pandoc.md

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

I have made some changes to this file.
The most important ones are:

* that I used `"$0"` instead of `.`
    * To give it the ability to run from every path
* I changed `magick` to `convert`
    * because the latest version that I could install on Docker was `6.9`

But unfortunately, my thought that these two can work separately didn't work.
So, at first, I changed the `destination_path` to generate PDFs in `static` instead of `public`.
Then, I observed that when `public` folder is being uploaded for `GitHub Pages`, PDFs can't be found.
Also, the process of installing **LaTeX** on Ubuntu Docker takes too long.

## Make a container for pandoc

To fix the problem of installing **Latex** each time, I thought it would be a good idea to
pre-build a Docker image and upload it.
So, I only pull that image instead of downloading and building **LaTeX** every time.
After some research, I came up with the `Dockerfile` with the content below:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y \
        pandoc \
        texlive-latex-base \
        texlive-fonts-recommended \
        texlive-fonts-extra \
        texlive-latex-extra \
        texlive-xetex \
        librsvg2-bin \
        imagemagick && \
    rm -rf /var/lib/apt/lists/*

CMD ["tail", "-f", "/dev/null"]
```

This `Dockerfile` installs everything needed for me.
Now, I only need to build it and upload it somewhere that I can use.
I found out that `GHCR` (`GitHub Container Registry`) is the best place.
So, I built my image using the code below:

```shell
docker build -t ghcr.io/literallytheone/pandoc-builder:latest .
```

Then I logged in and uploaded my Docker image using the code below:

```shell
echo $GITHUB_TOKEN | docker login ghcr.io -u literallytheone --password-stdin
docker push ghcr.io/literallytheone/pandoc-builder:latest
```

Now, I should change my workflow to use this container.
I can do this as follows:

```yaml
runs-on: ubuntu-latest
container:
  image: ghcr.io/literallytheone/pandoc-builder:latest
```

The problem that I have right now is that it can be `pulled` by `GitHub Actions`, but
it says that it is not running correctly.
I don't have a problem running it with `act` on my MacBook.
So, I should debug it tomorrow.

## Final thoughts

`Github Actions` is a super useful tool for `CI/CD`.
With `act`, we can test our `Workflow` before using `GitHub Action`, which eliminates so
many bugs and saves so much time.
I was installing so many packages in the `ubuntu:latest` container, which was taking so long.
I decided to use a pre-built container, and I faced some bugs in `Github Actions` which I'm going to resolve
as soon as possible.


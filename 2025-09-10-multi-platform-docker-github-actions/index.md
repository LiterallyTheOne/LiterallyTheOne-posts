---
date: '2025-09-10T14:05:00+03:30'
draft: false
title: "Build a multi-platform image with Docker and fix the GitHub Workflow"
description: "A post about how to build a multi-platform image with Docker and how I fixed and optimized GitHub Workflow"
tags: [ "Pandoc", "pdf", "latex", "hugo", "GitHub", "github-actions", "docker" ]
image: "multi-platform-image-github-workflow.webp"
---

# Build a multi-platform image with Docker and fix the GitHub Workflow

## Introduction

Today, I planned to fix the bug that I had with **GitHub Workflow**.
My bug was that my generated image wouldn't run on **GitHub Workflow**, but was working fine
on my MacBook.
I found out that the problem was because of the different platforms that my MacBook and GitHub have.
I was building my image with `arm64`, but GitHub expected it to be `amd64`.
After fixing that bug and publishing it, I optimized my workflow as well.

## Multi-platform build with Docker

To solve the bug that I was facing, I searched for how to build my image with Docker in
multi-platform.
After some research, I found out that I should enable `Use containerd for pulling and storing`.
I was using **Docker Desktop**, so to do that, I should have enabled it in the **general** section of
**settings**.
After that, I changed my `Dockerfile` to include `git` and run `bash`.
Also, I have removed the LaTeX extra fonts package, which would have taken so much space.
So, my `Dockerfile` looks like the following:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y \
        pandoc \
        texlive-latex-base \
        texlive-fonts-recommended \
        texlive-latex-extra \
        texlive-xetex \
        librsvg2-bin \
        imagemagick \
        git \
    && \
    rm -rf /var/lib/apt/lists/*


CMD ["/bin/bash"]
```

Then, I built my image using the code below:

```shell
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/literallytheone/pandoc-builder:latest .
```

With the code above, I would have 2 versions of my image.
One with `amd64` to run it on **GitHub Workflow** and one with `arm64` to run it on my MacBook.

## Fix the GitHub Workflow

Now, it was time to fix and optimize the **GitHub Workflow**.
First, I had to make some changes to my PDF converter codes.
I changed them in a way that they produce their output in the `public` folder instead of the `static` folder.
The reason for that was that generating PDFs and generating a Hugo website can be done simultaneously, and their
output can be combined afterward.
So, I used `artifact` to store their outputs, and I combined them in the `deploy` step.
My **GitHub Workflow** looks like the following:

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Build PDFs using Pandoc, Build and Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    # GitHub-hosted runners automatically enable `set -eo pipefail` for Bash shells.
    shell: bash

jobs:

  build-pdf:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/literallytheone/pandoc-builder:latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Convert each md to pdf
        run: |
          bash convert_to_pdf_pandoc/convert_each_md_to_pdf.sh

      - name: Convert each tutorial to pdf
        run: |
          bash convert_to_pdf_pandoc/convert_tutorial_to_book.sh

      - name: Upload PDF artifact
        uses: actions/upload-artifact@v4
        with:
          name: pdfs
          path: site/public/pdf

  build-hugo:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.89.2
      HUGO_VERSION: 0.148.0
      HUGO_ENVIRONMENT: production
      TZ: America/Los_Angeles
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: |
          wget -O ${{ runner.temp }}/dart-sass.tar.gz https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz
          tar -xf ${{ runner.temp }}/dart-sass.tar.gz --directory ${{ runner.temp }}
          mv ${{ runner.temp }}/dart-sass/ /usr/local/bin
          echo "/usr/local/bin/dart-sass" >> $GITHUB_PATH

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"

      - name: Cache Restore
        id: cache-restore
        uses: actions/cache/restore@v4
        with:
          path: |
            ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-

      - name: Configure Git
        run: git config core.quotepath false

      - name: Build with Hugo
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
        working-directory: site

      - name: Cache Save
        id: cache-save
        uses: actions/cache/save@v4
        with:
          path: |
            ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}

      - name: Upload site artifact
        uses: actions/upload-artifact@v4
        with:
          name: site
          path: site/public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: [ "build-hugo", "build-pdf" ]
    steps:
      - name: Download Site
        uses: actions/download-artifact@v4
        with:
          name: site
          path: site/public

      - name: Download PDFs
        uses: actions/download-artifact@v4
        with:
          name: pdfs
          path: site/public/pdf

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site/public

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

As you can see, in the code above, I have 3 jobs: `build-pdf`, `build-hugo`, and `deploy`.
`build-pdf` and `build-hugo` generate their output and put them in their respective `artifact`.
Then, `deploy` waits for them to be finished.
After that, it combines the outputs and deploys them on **GitHub pages**. 

## Final thoughts

Building a multi-platform Docker image helps our image run on different environments.
In my scenario, I had to make it with `amd64`.
Also, separating building PDFs and building the site helps the process of deploying faster. 
Now, by clicking on the **Download PDF** button, you can download the PDF of each post and tutorial.
Also, with the **Download book** button**, you can download all the pages of a tutorial.

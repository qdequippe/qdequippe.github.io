# qdequippe.github.io

Personal website built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

- **Main URL**: `https://dequippe.tech`
- **Languages**: French (`fr`) and English (`en`)
- **Configuration**: `hugo.yaml`

## Prerequisites

- Git
- Hugo **Extended** (recommended)

Check your installation:

```bash
git --version
hugo version
```

## Installation

Clone the repository, then initialize the theme submodule:

```bash
git clone https://github.com/qdequippe/qdequippe.github.io.git
cd qdequippe.github.io
git submodule update --init --recursive
```

## Local Development

Start the Hugo server with drafts enabled (`draft: true`):

```bash
hugo server -D
```

Then open:

- `http://localhost:1313/`
- `http://localhost:1313/fr/`
- `http://localhost:1313/en/`

## Production Build

Generate the static site into `public/`:

```bash
hugo --minify
```

The `public/` folder is generated automatically and ignored by Git (`.gitignore`).

## Project Structure

```text
.
|- hugo.yaml                  # Global site config (theme, languages, menus, params)
|- content/
|  `- posts/                  # Blog posts
|- archetypes/
|  `- default.md              # Default front matter for new content
|- static/
|  `- images/                 # Static assets served as-is
|- themes/
|  `- PaperMod/               # Theme Git submodule
`- public/                    # Generated site (build)
```

## Add a Post

1. Create a new post:

```bash
hugo new content posts/my-new-post.md
```

2. Edit the generated file in `content/posts/`.
3. Set `draft: false` when the post is ready.
4. Run `hugo server -D` to preview locally.

Example front matter (based on existing content):

```yaml
---
date: 2026-04-14
title: My new post
linktitle: My new post
type:
  - post
  - posts
tags:
  - hugo
  - symfony
language: "English"
description: "Short summary of the post"
images:
  - /images/mon-image.jpg
draft: true
---
```

## Deployment

Deployment consists of publishing the generated files in `public/` to a static host (GitHub Pages, Netlify, web server, etc.).

Typical flow:

```bash
hugo --minify
# then publish the content of ./public based on your platform
```

## Useful Notes

- The `PaperMod` theme is a Git submodule: update it when needed.
- Multilingual configuration and menus are defined in `hugo.yaml`.
- Post images can be stored in `static/images/`.





# Thaddeuskoenig.com


Source repository for my personal website and technical blog.

The site is primarily a place for me to document projects, experiments, and the various technical rabbit holes I end up going down. The goal is to keep useful documentation somewhere searchable and public instead of leaving it scattered across notes, shell history, and old terminal windows.

The site is built entirely with open-source tooling using [Hugo](https://gohugo.io/) and the [hugo-theme-gruvbox](https://github.com/schnerring/hugo-theme-gruvbox) theme. The generated static site is automatically built and hosted using GitHub Actions and GitHub Pages.

## How It Works

At a high level, the site uses:

- **Hugo** as the static site generator
- **hugo-theme-gruvbox** as a Hugo Module
- **Go Modules** to manage Hugo theme dependencies
- **Node.js / npm** for the theme's frontend build dependencies
- **GitHub** for source control
- **GitHub Actions** to build the site
- **GitHub Pages** to host the generated static files

When a change is merged into `main`, GitHub Actions builds the site and deploys the resulting `public/` directory directly to GitHub Pages.

## Getting Started

You will need the following installed locally:

- Git
- Hugo Extended
- Go
- Node.js and npm

Clone the repository:

```bash
git clone https://github.com/adminprivileges/thaddeuskoenig.com.git
cd thaddeuskoenig.com
```

Install the Node.js dependencies:

```bash
npm ci --install-strategy=hoisted
```

Hugo will download the required Hugo Modules automatically when needed.

To confirm everything is working:

```bash
hugo server
```

The development site will normally be available at:

```text
http://localhost:1313/
```

## Creating a Blog Post

Blog posts are stored under:

```text
content/blog/
```

Each post is kept as a Hugo page bundle:

```text
content/blog/my-new-post/
└── index.md
```

This makes it easy to keep any images or other files associated with a post alongside the Markdown document.

Create a new post with Hugo:

```bash
hugo new content blog/my-new-post/index.md
```

You can also create the directory and Markdown file manually:

```bash
mkdir -p content/blog/my-new-post
vim content/blog/my-new-post/index.md
```

A post will generally begin with front matter similar to:

```yaml
---
title: "My New Post"
date: 2026-08-11
draft: true
tags:
  - linux
  - documentation
---
```

Write the article below using normal Markdown.

## Testing a New Post

While writing, start the Hugo development server with drafts enabled:

```bash
hugo server -D
```

Open:

```text
http://localhost:1313/
```

Hugo watches the repository for changes and automatically rebuilds the site as files are edited.

Before publishing, change:

```yaml
draft: true
```

to:

```yaml
draft: false
```

Then test a production-style build:

```bash
HUGO_ENVIRONMENT=production hugo --gc --minify
```

The command should complete successfully without build errors.

The generated website will be placed in:

```text
public/
```

The `public/` directory is ignored by Git because GitHub Actions generates it again during deployment.

## Publishing Changes

I generally make changes on a separate Git branch rather than directly on `main`.

Start from an up-to-date copy of `main`:

```bash
git switch main
git pull
```

Create a branch for the change:

```bash
git switch -c blog/my-new-post
```

Create or edit the post and test it locally:

```bash
hugo server -D
```

Before committing, run a production build:

```bash
HUGO_ENVIRONMENT=production hugo --gc --minify
```

Review the changes:

```bash
git status
git diff
```

Stage and commit them:

```bash
git add .
git commit -m "Add my new post"
```

Push the branch to GitHub:

```bash
git push -u origin blog/my-new-post
```

Then open a Pull Request from the new branch into `main`.

After reviewing the changes, merge the Pull Request.

A merge into `main` automatically triggers the GitHub Actions deployment workflow:

```text
Local changes
     ↓
Git branch
     ↓
GitHub Pull Request
     ↓
Merge to main
     ↓
GitHub Actions
     ↓
Hugo production build
     ↓
GitHub Pages
```

There is no need to manually build or commit the contents of `public/`.

## Updating the Hugo Theme

The Gruvbox theme is installed as a Hugo Module rather than being copied directly into the repository.

The currently installed modules can be inspected with:

```bash
hugo mod graph
```

Theme updates should be performed deliberately rather than automatically updating everything during deployment.

After changing Hugo Module versions, clean the module metadata:

```bash
hugo mod tidy
```

Regenerate Hugo's npm dependency workspace:

```bash
hugo mod npm pack
```

Then reinstall the Node dependencies:

```bash
rm -rf node_modules
npm install --install-strategy=hoisted
```

Finally, test the site:

```bash
hugo server
```

and perform a production build:

```bash
HUGO_ENVIRONMENT=production hugo --gc --minify
```

Dependency updates should be tested locally before being merged into `main`.

## Repository Structure

Some of the important directories and files are:

```text
.github/workflows/     GitHub Actions build/deployment workflows
archetypes/            Hugo templates for new content
assets/                Site assets processed by Hugo
content/               Markdown content
content/blog/          Blog posts
content/projects/      Project pages
data/                  Structured site data
layouts/               Local Hugo template overrides
static/                Files copied directly into the generated site
hugo.toml              Main Hugo configuration
go.mod / go.sum        Hugo Module dependency versions
package.json           Node project configuration
package-lock.json      Reproducible Node dependency lockfile
packages/hugoautogen/  Dependencies generated from Hugo Modules
```

The following are generated locally and should not be committed:

```text
node_modules/
public/
resources/_gen/
.hugo_build.lock
hugo_stats.json
```

## Why Hugo?

Hugo works well for this site because the source material is just Markdown and configuration files. There is no database, application server, or content management system, also its free.

The final website consists of static HTML, CSS, JavaScript, images, and other assets that can be served directly by GitHub Pages.

That keeps the site relatively simple, portable, inexpensive to host, and easy to reproduce.

The entire site can effectively be rebuilt from this repository with:

```bash
git clone https://github.com/adminprivileges/thaddeuskoenig.com.git
cd thaddeuskoenig.com

npm ci --install-strategy=hoisted
hugo server
```

For the most part, if this repository exists, the website can be rebuilt.

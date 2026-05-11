+++
title       = "How I built this site for free (and how you can too)"
date        = 2026-05-07
draft       = false
tags        = ["hugo", "github pages", "blogging", "career"]
description = "A beginner-friendly guide to building a free cybersecurity blog using Hugo and GitHub Pages — no coding experience required."
+++

This site costs nothing to run. No hosting fees, no WordPress subscription, no website builder. It's built with Hugo, hosted on GitHub Pages, and served on a custom domain I pay about $12 a year for.

If you're in cybersecurity or IT and want a blog that doubles as a portfolio, this is the stack I'd recommend. Here's how to build one from scratch — even if you've never used a terminal before.

## What you'll be building

A static website that looks professional, loads fast, and deploys automatically every time you publish a new post. You write in plain text, run one command, push to GitHub, and it's live.

The whole setup takes a few hours the first time. After that, publishing a post takes under five minutes.

## What you'll need

- A computer running Mac, Windows, or Linux
- A free GitHub account
- A domain name (optional — you get a free one from GitHub, but a custom domain looks more professional)

No prior coding experience is required. You'll be using the terminal, but every command is explained.

---

## Step 1 — Install Git

Git is a tool that tracks changes to your files and lets you push them to GitHub. Think of it as a save system for code.

Download it from [git-scm.com](https://git-scm.com) and run the installer. Once it's installed, open your terminal (search "Terminal" on Mac, "Command Prompt" or "PowerShell" on Windows) and run:

```bash
git --version
```

If you see a version number, Git is installed.

---

## Step 2 — Install Hugo

Hugo is the engine that turns your text files into a website. It's fast, free, and widely used.

**Mac:**
```bash
brew install hugo
```
(If you don't have Homebrew, install it first at [brew.sh](https://brew.sh))

**Windows:**
```bash
winget install Hugo.Hugo.Extended
```

**Linux:**
```bash
sudo apt install hugo
```

Confirm it worked:
```bash
hugo version
```

---

## Step 3 — Create your site

Navigate to where you want your site folder to live. Your home directory is fine:

```bash
cd ~
hugo new site mysite
cd mysite
```

This creates a folder called `mysite/` with everything Hugo needs. It's empty for now — no theme, no content yet.

---

## Step 4 — Install a theme

A theme controls how your site looks. Hugo has hundreds of free themes at [themes.gohugo.io](https://themes.gohugo.io). This guide uses PaperMod — it's clean, fast, and works great for technical blogs.

First, initialise Git in your site folder:
```bash
git init
```

Then install PaperMod:
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

---

## Step 5 — Configure your site

Open the `hugo.toml` file in your site folder with any text editor (Notepad, TextEdit, or VS Code). Replace everything in it with this:

```toml
baseURL = "https://yourusername.github.io/mysite/"
title   = "Your Site Title"
theme   = "PaperMod"

[params]
  defaultTheme = "auto"

  [params.homeInfoParams]
    Title   = "Your Name"
    Content = "A short description of what your site is about."

  [[params.socialIcons]]
    name = "github"
    url  = "https://github.com/yourusername"

[[menu.main]]
  name   = "About"
  url    = "/about/"
  weight = 10
```

Replace `yourusername` with your actual GitHub username and fill in your title and description.

---

## Step 6 — Write your first post

Hugo has a built-in command to create new posts:

```bash
hugo new content posts/my-first-post.md
```

This creates a file at `content/posts/my-first-post.md`. Open it in a text editor. You'll see something like this at the top:

```
+++
title = "My First Post"
date  = 2026-01-01
draft = true
+++
```

Change `draft = true` to `draft = false`, then write your post below the `+++` line using plain text. You can use Markdown for formatting — `**bold**`, `# Heading`, `` `code` ``, and so on.

---

## Step 7 — Preview locally

Before publishing, you can see exactly how your site looks on your own computer:

```bash
hugo server -D
```

Open `http://localhost:1313` in your browser. Your site is running locally — no internet required. Any changes you save will appear instantly.

Press `Ctrl + C` to stop the server when you're done.

---

## Step 8 — Create a GitHub repo

Go to [github.com/new](https://github.com/new) and create a new public repository. Name it `mysite` (or whatever you called your folder). Don't tick "Add a README".

---

## Step 9 — Set up auto-deploy

This is the part that makes GitHub automatically rebuild your site every time you push. Create this folder structure inside your site folder:

```
.github/
  workflows/
    hugo.yml
```

On Mac and Linux you can do this in the terminal:
```bash
mkdir -p .github/workflows
```

Then create a file called `hugo.yml` inside that folder and paste in exactly this:

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: latest
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    env:
      FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## Step 10 — Push to GitHub

Run these commands in your site folder, replacing the URL with the one GitHub gave you when you created the repo:

```bash
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/mysite.git
git push -u origin main
```

---

## Step 11 — Enable GitHub Pages

In your GitHub repo go to **Settings → Pages → Source** and select **GitHub Actions**.

Go to the **Actions** tab — you'll see a workflow running. Wait for the green checkmark. Your site is now live at `https://yourusername.github.io/mysite/`.

---

## Step 12 — Connect a custom domain (optional)

If you want your own domain instead of the default GitHub URL, buy one from a registrar like Namecheap or GoDaddy — `.com` domains run about $10–15 a year.

Once you have it, go to your repo → **Settings → Pages → Custom domain** and enter your domain.

Then log into your registrar and add these DNS records:

| Type | Name | Value |
|------|------|-------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | yourusername.github.io |

DNS changes can take a few minutes to a few hours to propagate. Once GitHub shows the DNS check as passing, tick **Enforce HTTPS**.

Finally update `hugo.toml` to use your new domain:
```toml
baseURL = "https://yourdomain.com/"
```

Push that change and you're done.

---

## Publishing new posts

From this point on, publishing is three commands:

```bash
git add .
git commit -m "new post"
git push
```

GitHub Actions rebuilds and deploys your site automatically. It's live within a minute.

---

## Why bother?

If you're job hunting in cybersecurity or IT, a blog is one of the highest-leverage things you can do. It shows you can communicate technical concepts, demonstrates that you're actively learning, and gives employers something to read beyond a resume.

The setup is a talking point in itself — you used Git, GitHub Actions, DNS configuration, and a static site generator. That's a non-trivial technical project, and it's yours.

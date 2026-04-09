## theaiglet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Aiglet is a satirical news site (Hugo static site) deployed to GitHub Pages at theaiglet.top. Think *The Onion* meets *The Atlantic*.

## Common Commands

```bash
hugo server              # Local dev server at localhost:1313
hugo server -D           # Include draft posts
hugo --gc --minify       # Production build (what CI runs)
git submodule update --init --recursive  # Initialize theme
```

There is no test suite or linter. Deployment is automatic via GitHub Actions on push to `main`.

## Architecture

- **Hugo static site** using the Quint theme (git submodule in `themes/quint/`)
- **Content**: `content/posts/YYYY-MM-DD-slug/index.md` — each article is a page bundle with its featured image
- **Custom layouts**: `layouts/` overrides theme templates (`_default/`, `partials/`, `index.html`)
- **Custom styles**: `static/css/custom.css`
- **Config**: `hugo.toml`

## Article Frontmatter Format

Frontmatter uses TOML (`+++` delimiters):

```toml
+++
date = '2026-01-17T14:30:00-05:00'
type = 'posts'
draft = false
title = 'Article Title'
slug = 'article-slug'
image = 'featured-image.jpg'
alt = 'Accessibility description'
tags = ['satire', 'technology']
description = "SEO/social description"
img-prompt = "AI image generation prompt for reproducibility"
+++
```

## Content Workflow

The primary workflow uses Claude Code agents: `satirical-article` (ideation/drafting), `hugo-publisher` (file creation), and `roast-my-writing` (critique). Article ideas live in `/home/greg/dev/ai-workshop/sessions/satirical-articles/ideas.md`.

## Editorial Voice

Deadpan journalistic delivery. Sharp specificity with concrete details (names, numbers, quotes). Let absurdity speak for itself — never over-explain the joke. Avoid punching down.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bravegeek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bravegeek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

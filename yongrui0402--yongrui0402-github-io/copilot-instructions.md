## yongrui0402-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Jekyll-based personal blog (YongRui's Blog) hosted on GitHub Pages at `https://YongRui0402.github.io/`. Content is primarily in Traditional Chinese.

## Development Commands

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve

# Build the site
bundle exec jekyll build
```

The site is deployed automatically via GitHub Pages on push to `main`.

## Architecture

**Jekyll collections and content types:**
- `_posts/` — Blog posts organized by category subdirectories (e.g., `Python/2024/`, `Other/2024/`). Posts use `layout: post`.
- `_wiki/` — Reference/cheatsheet pages (e.g., Git commands). Use `layout: wiki`.
- `_tool/` — Interactive tool pages with embedded HTML/JS (e.g., calculators). Use `layout: tool`.
- `pages/` — Static pages (about, categories, links, wiki index, 404).

**Post filename convention:** `YYYY-MM-DD-title.md` with front matter: `layout`, `title`, `categories`, `description`, `keywords`.

**Layout hierarchy:** `_layouts/default.html` wraps all pages (includes `header.html` + `footer.html`). Specific layouts (`post`, `wiki`, `tool`, `categories`, `page`) extend `default`.

**Data files (`_data/`):** `links.yml` (blogroll), `skills.yml` (skills list), `social.yml` (social links) — consumed by layouts/includes.

**Client-side features:** Search via `jekyll-search.min.js` + `search_data.json`, syntax highlighting via Prism.js, header patterns via GeoPattern.js.

**Config notes (`_config.yml`):**
- Pagination: 8 posts per page via `jekyll-paginate`
- Markdown: kramdown with GFM input, rouge highlighter
- Comments: Disqus (primary), with gitment/gitalk alternatives
- Permalink format: `/:year/:month/:day/:title/`
- Custom domain configured in `CNAME`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YongRui0402)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YongRui0402)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

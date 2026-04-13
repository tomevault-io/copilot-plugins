## fbeeper-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bundle install          # Install Ruby gem dependencies
bundle exec jekyll serve         # Local dev server (http://localhost:4000)
bundle exec jekyll serve --drafts  # Include drafts in dev server
bundle exec jekyll build         # Build to _site/
```

The `_site/` directory is the build output — don't edit files there directly.

## Architecture

This is a Jekyll static site deployed on GitHub Pages at `www.fbeeper.com`.

**Layout hierarchy:**
- `_layouts/default.html` — outer wrapper; includes `head.html` and `sidebar.html`, then processes content through `anchor_headings.html`
- `_layouts/post.html` — blog post view; uses `time.html` (reading time) and `image.html`
- `_layouts/page.html` — static page view
- `_layouts/filter.html` — category/tag archive view with `pagination.html`

**CSS loading order** (all in `_includes/head.html`):
1. `theme.css` — CSS custom properties for colors and theming (dark/light modes via `prefers-color-scheme`)
2. `poole.css` — base reset and typography
3. `hyde.css` — sidebar layout
4. `fbeeper.css` — site-specific overrides and additions
5. `syntax.css` — Rouge code highlighting
6. `decorative.css` — the polygon/shape decorations on the sidebar

**Post front matter conventions:**
```yaml
---
layout: post
title: "Post Title"
categories: [general]  # e.g. general, development, accessibility, comphist, blogging
tags: [tag1, tag2]
image: /public/img/filename.jpg  # optional, used for Twitter card
description: "Short description"  # optional, used for SEO
---
```

**Decorative shapes** (`_config.yml` → `decoration` key): The sidebar background uses CSS clip-path polygons defined in `decorative.css`. The active decoration is set per-page via front matter or site-wide in `_config.yml`.

**Analytics**: GoatCounter is used for privacy-respecting analytics via an image pixel (no JavaScript). Configuration is in `_includes/head.html`.

**No JavaScript** is used on the site — it's intentionally pure HTML/CSS.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fbeeper)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fbeeper)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

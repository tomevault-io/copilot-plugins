## blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A custom static blog generator for waju.dev. Node.js build script compiles MDX posts into static HTML, generates OG images, and deploys to Cloudflare Pages.

## Commands

```bash
npm run build          # Generate site into dist/
npm run deploy         # Build + deploy to Cloudflare Pages
```

To preview locally:
```bash
cd dist && python3 -m http.server 3000
```

## Architecture

**Single build script (`build.js`)** handles everything:
1. Copies `static/` assets to `dist/`
2. For each `.mdx` file in `posts/`: parses frontmatter (gray-matter), compiles MDX to HTML (via `@mdx-js/mdx` evaluate + React `renderToStaticMarkup`), applies `templates/post.html` via `{{PLACEHOLDER}}` string replacement, generates a 1200x630 OG PNG (satori → @resvg/resvg-js)
3. Generates `index.html` from `templates/home.html` with posts grouped by year
4. Generates `rss.xml` and `og-home.png`

**No framework, no template engine** — just string replacement on HTML templates with `{{TITLE}}`, `{{CONTENT}}`, `{{SLUG}}`, etc.

## Adding a New Post

Create `posts/my-post-slug.mdx`:
```yaml
---
title: "Post Title"
date: 2025-01-15
description: "For meta tags and RSS"
---

MDX content here. Code blocks get syntax highlighting via rehype-highlight.
```

The filename (minus `.mdx`) becomes the URL slug: `/posts/my-post-slug`.

## Key Files

- `build.js` — The entire build pipeline (MDX compilation, templating, OG generation, RSS)
- `templates/home.html` — Homepage template (`{{POSTS}}` placeholder)
- `templates/post.html` — Post template (11 placeholders including `{{TITLE}}`, `{{CONTENT}}`, `{{SLUG}}`, `{{DATE_LONG}}`, `{{DATE_ISO}}`, `{{READ_TIME}}`)
- `static/style.css` — All styles; CSS variables for light/dark theming (`[data-theme="dark"]`)
- `static/SpaceMono-*.ttf` — Font files used by satori for OG image generation
- `wrangler.jsonc` — Cloudflare Pages config (output dir: `./dist`)

## Design System

- Font: Space Mono (monospace)
- Content width: 720px
- Light mode: `#FFFFFF` bg, `#1A1A1A` text, `#333333` secondary, `#555555` tertiary
- Dark mode: `#111111` bg, `#F0EDED` text, `#A0A0A0` secondary, `#666666` tertiary
- Accent: `#C4553A`
- Mobile breakpoint: 768px (shows "Waju" instead of "Waju Abolarin", stacked post items)

## Theming

Dark/light mode uses `data-theme="dark"` on `<html>`. An inline script in `<head>` reads `localStorage('theme')` or `prefers-color-scheme` before paint to prevent flash. Toggle button in header persists choice.

## Deployment

Connected to GitHub (`WajuAbolarin/blog`). Pushes to `main` trigger Cloudflare Pages builds automatically. Build command: `npm run build`, output: `dist/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WajuAbolarin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WajuAbolarin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

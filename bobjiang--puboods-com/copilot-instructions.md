## puboods-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo static site for puboods.com — a software development company website. Uses the `pehtheme-hugo` theme (Tailwind CSS, blog-oriented) and is hosted on Cloudflare.

## Setup

After cloning, initialize the theme submodule and install dependencies:

```
git submodule update --init --recursive
npm install
```

## Development Commands

- `npm run dev` — Dev mode with CSS watch + Hugo server (preferred for development)
- `hugo server` — Dev server with live reload (no CSS rebuilding)
- `hugo server -D` — Dev server including draft content
- `npm run build` — Full production build (CSS + Hugo)
- `npm run build:css` — Rebuild Tailwind CSS only

## Architecture

### Configuration

Hugo automatically merges all `config*.toml` files. The split:
- **hugo.toml** — Primary config: theme, menus, params, author info, taxonomies. Uses `baseURL = "/"` for local dev.
- **config.toml** — Production overrides: sets `baseURL = 'https://puboods.com/'` and meta description. Values here win over `hugo.toml`.

### Theme

Git submodule at `themes/pehtheme-hugo` (Tailwind CSS, blog-oriented). The theme provides:
- `home.html` — Homepage with featured post hero + category section + recent posts grid
- `page/single.html` — Static pages with featured image + title + prose content + sidebar
- `_default/single.html` — Blog posts with author, date, related posts
- `_default/list.html` — Archive/category listing pages

### Layout Overrides

Only two custom layout files override the theme:
- **`layouts/_default/home.html`** — Changes the category section from "astronomy" to "projects"
- **`layouts/partials/footer.html`** — Replaces theme's placeholder text with Puboods branding

### Content Structure

```
content/
  _index.md          — Homepage (uses home.html)
  services.md        — Services page (rendered by page/single.html)
  industries.md      — Industries page
  portfolio.md       — Portfolio page
  contact.md         — Contact page
  posts/             — Blog posts (populate homepage grid)
    apex-omni-case-study.md       — Featured project (tagged "feature" for hero)
    hyperindex-case-study.md      — Project case study
    our-services-overview.md      — Services overview post
    client-success-stories.md     — Testimonials post
```

Posts with the `feature` tag appear in the homepage hero section. Posts with the `projects` category appear in the homepage category section.

### Static Assets

- `static/img/logo.svg` — Site logo
- `static/img/fav.svg`, `fav.png` — Favicons
- `static/img/portfolio/` — Portfolio screenshots (apex-omni.jpg, hyper-index.jpg)

### Deployment

Hosted on Cloudflare. Build output goes to `public/`.

### Tailwind CSS

CSS is built from `assets/input.css` into `themes/pehtheme-hugo/assets/css/main.css`. Tailwind scans `content/**/*.md`, `layouts/**/*.html`, and `themes/pehtheme-hugo/**/*.{html,js}` for class usage (configured in `tailwind.config.js`). Run `npm run build:css` after adding new Tailwind classes, or use `npm run dev` which watches for changes automatically.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bobjiang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

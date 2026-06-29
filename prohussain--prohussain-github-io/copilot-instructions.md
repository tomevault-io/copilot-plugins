## prohussain-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static portfolio website for Ghulam Hussain, a Principal Android Engineer. It is a single-page application (SPA) built with plain HTML, CSS, and vanilla JavaScript — no build tools, no package manager, no framework.

To preview the site, open `index.html` directly in a browser or use any static file server:
```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Architecture

All content is in a single file: `index.html`. The page is organized into these sections (in order):
- Header + sidebar navigation
- Hero (intro + social links)
- Impact Metrics (key numbers)
- Skills (`#skills`)
- Portfolio / Projects (`#portfolio`)
- Experience (`#experience`)
- Education (`#education`)
- Features / Services
- Blog & Articles
- Footer

### CSS Structure

Styles are split across multiple files loaded in order in `<head>`:

| File | Purpose |
|------|---------|
| `css/global.css` | CSS variables, resets, shared components (headings, buttons, social links). The neumorphic design system lives here. |
| `css/styles.css` | Header, sidebar, hero, and core section layouts |
| `css/impact-metrics.css` | Metric cards section |
| `css/principal-enhancements.css` | Skills grid, project cards, timeline, blog cards |
| `css/queries.css` | All responsive breakpoints (mobile-first) |
| `css/animations.css` | CSS animation keyframes and `.animated` / `animate-on-scroll` classes |
| `css/testimonials.css` | Testimonial card styles (section currently unused in HTML) |
| `css/services-contact.css` | Contact/services section styles |

### Design System (from `css/global.css`)

- **Primary color**: `#0D47A1` (deep blue) via `--color-primary`
- **Background**: `#ecf0f3` (neumorphic base)
- **Neumorphic shadow**: `5px 5px 15px #d1d9e6, -5px -5px 15px #ffffff`
- **Font**: Poppins (Google Fonts), base size 62.5% (1rem = 10px)
- **Spacing scale**: 2/4/8/12/16/24/32/48/64/80/96/128 px

### JavaScript

| File | Purpose |
|------|---------|
| `js/scripts.js` | Sidebar open/close, scroll-to-top button |
| `js/animations.js` | IntersectionObserver-based scroll animations for all sections; skill bar counters; smooth nav scroll |
| `js/contact-form.js` | Contact form handling |

Animations are triggered by adding `.animated` or `.animate` CSS classes via `IntersectionObserver` when elements enter the viewport. The `animations.js` also disables animations on low-end devices (`hardwareConcurrency < 4`) and low battery.

## Key Conventions

- All content changes go in `index.html` — sections, project cards, experience entries, etc.
- New sections must be wrapped in `<section>` and separated by `<div class="seperator">&nbsp;</div>` (note the typo: "seperator" not "separator" — keep consistent).
- Project cards follow the `.project-card > .project-card--head + .project-card--body` structure.
- Timeline entries use `.timeline-step > .timeline-content + .timeline-arrow` (no `.timeline-arrow` on last step).
- Images go in `images/portfolio-imgs/` for projects, `images/` for general assets. Use `.webp` where possible.
- No build step exists — edits take effect immediately on page refresh.

---
> Source: [ProHussain/prohussain.github.io](https://github.com/ProHussain/prohussain.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

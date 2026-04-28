## shubby98-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static single-page portfolio website for Shubham Shakya (AI Engineer). No build system, no package manager, no framework — pure HTML/CSS/ES Modules. Open `index.html` directly in a browser or serve with any static file server.

## Development

To preview locally:
```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

There are no lint, test, or build commands.

## Architecture

Everything lives in three files + a JS module tree:

- **`index.html`** — All page content and structure. Contains an inline PostHog analytics snippet in `<head>`. Loads `js/main.js` as `type="module"`.
- **`style.css`** — All styles for the entire site.
- **`js/main.js`** — Entry point; imports and initializes all modules on `DOMContentLoaded`.

### JS Modules (`js/modules/`)

| File | Purpose |
|---|---|
| `particles.js` | Canvas-based animated particle network rendered on `#hero-canvas` |
| `animations.js` | `IntersectionObserver` scroll-reveal (adds `.visible` class) |
| `navigation.js` | Smooth scroll for `a[href^="#"]` links with 80px navbar offset |
| `ui.js` | Mobile menu stub (currently a no-op `console.log`) |

### Page Sections (in order)

1. **Hero** (`#home`) — avatar, name, title, CTA buttons, social links, canvas particle background
2. **Split layout** (`#split-content`) — sticky left aside (Professional Summary + Technical Expertise skills) + scrollable right (timeline of education/work entries)
3. **Projects** (`#projects`) — bento-grid of 6 project cards (2 `large`, 4 `medium`)
4. **Footer/Contact** (`#contact`) — social icon links, copyright

### Key CSS Patterns

- `.bento-grid` / `.bento-item.large` / `.bento-item.medium` — project card grid
- `.timeline-split` / `.timeline-entry.left.education` / `.timeline-entry.right.work` — two-column timeline
- `.reveal` class on hero-container triggers entry animation
- `.visible` class added by `animations.js` to reveal elements on scroll

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shubby98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->

## gem-chapel-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GEM Chapel** (Great Expectations Ministries International) — a static, single-page church website deployed on Vercel. No build system, package manager, or test framework.

## Architecture

Three source files:
- **`index.html`** — semantic HTML structure with section landmarks
- **`styles.css`** — all CSS including design tokens (`:root` variables), responsive breakpoints, and animations
- **`script.js`** — IIFE-wrapped vanilla JS for interactivity

### HTML Sections (in order)
Navigation → Hero → About → Activities → Leadership → Resources (Articles) → Books → Branches → Donate → Contact → Footer

Decorative `section-divider` elements separate major content groups.

### CSS Design System (`:root` variables)
- **Colors:** `--flame-gold`, `--flame-orange`, `--flame-deep`, `--flame-ember` (fire theme); `--dark-bg`, `--dark-surface`, `--dark-card` (backgrounds); `--cream`, `--cream-muted`, `--white` (text)
- **Fonts:** `--font-display` (Fraunces — headings), `--font-body` (Outfit — body), `--font-accent` (Cormorant Garamond — decorative/quotes)
- **Easing:** `--ease-out-expo`, `--ease-out-back` for consistent motion
- Responsive breakpoints at 1024px, 768px, and 480px

### JavaScript Features
- Scroll-triggered navbar with blur backdrop (`#navbar.scrolled`)
- Active nav link highlighting based on scroll position
- Animated hamburger toggle with full-screen mobile overlay (`.nav-links.open`)
- DOM-based flame particle system in hero (responsive particle count: 25 mobile, 50 desktop)
- IntersectionObserver-based scroll reveal (`.reveal` → `.visible`), elements unobserved after reveal
- Animated counters on `[data-count]` elements (years of ministry, branch count)
- Smooth anchor scrolling with navbar offset compensation
- Form validation with shake animation feedback — **client-side only, no backend submission**

### Key Patterns
- All inline SVG icons use `stroke`-based rendering with `fill: none` and `stroke-width: 1.5`
- Card hover effects use `transform: translateY(-Npx)` with `var(--ease-out-expo)` transitions
- Staggered reveal animations use `.reveal-delay-1` through `.reveal-delay-4` utility classes
- The `@keyframes shake` animation is inlined in `<head>` (used by JS form validation)

## Development

Open `index.html` directly in a browser, or use any static file server:
```
python3 -m http.server
```

## Deployment

Deployed via Vercel (project: `gem-chapel-website`). Zero-config static site auto-detected from `index.html`. No build step.

## Other Assets (git-ignored)

- **`Fresh Fire/`** — PDF devotional publications (~90MB)
- **`Logos/`** — JPG logo variants (GEM - Selected-03/04/05.jpg)
- **`GEM WEBSITES- UPDATED INFO.docx`** — Reference document with updated church info

**Not git-ignored but superseded:** `gem-chapel-website.html` — original single-file version, replaced by the split `index.html` + `styles.css` + `script.js` setup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reedrich12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/reedrich12)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## my-profile-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Site

No build step required. Open `index.html` directly in a browser, or serve locally:

```bash
python -m http.server 8000
```

## Architecture

Three files compose the entire site:

- **index.html** — Full single-page portfolio with all markup and inline Tailwind classes. Sections (in order): navbar, hero, tech strip, about, skills, projects, contact.
- **style.css** — Custom styles layered on top of Tailwind CDN: glassmorphism cards, gradient text/buttons, keyframe animations (`blink` for typing cursor, `float` for avatar), scroll-reveal transitions, and custom red scrollbar.
- **main.js** — Loaded at end of `<body>`. Five features:
  1. **Typing animation** — cycles through 5 developer role strings in the hero section
  2. **Navbar scroll effect** — adds `scrolled` class at >40px scroll
  3. **Active nav highlight** — updates nav link styles based on visible section via scroll listener
  4. **Scroll reveal** — `IntersectionObserver` on `.reveal` elements; adds `visible` class at 12% threshold
  5. **Mobile menu toggle** — hamburger open/close, auto-closes on link click

Tailwind CSS is loaded via CDN (no local install). The site is Korean-language with some English role strings in the typing animation.

---
> Source: [tjdals/my-profile-site](https://github.com/tjdals/my-profile-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->

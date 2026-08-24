## billykov-site

> Static HTML/CSS developer website for Billy Kovalsky. Serves two functions:

# Developer Website — Project Guide

## Purpose

Static HTML/CSS developer website for Billy Kovalsky. Serves two functions:
1. Personal developer presence (about me, apps built, contact)
2. Legal hosting for monday.com Marketplace compliance — Terms of Service and Privacy Policy for all published apps

This site must be live before any app can be submitted to the monday Marketplace.

## Tech Constraints

- **Static site** — HTML, CSS, and vanilla JS only; no backend, no server-side code
- No heavy frameworks — lightweight libraries (e.g. Alpine.js, Animate.css) are fine if needed
- No build tools required — deployable as-is to GitHub Pages, Netlify, or similar
- Must be accessible via a public HTTPS URL (required by monday Marketplace)
- Mobile responsive

## Site Structure

```
/
  index.html         — Landing page (about me, apps)
  tos.html           — Terms of Service (per app or unified)
  privacy.html       — Privacy Policy (per app or unified)
  css/
    style.css
  assets/
    (images, icons)
```

## Content Requirements

### index.html
- Developer name and brief bio
- List of published apps with descriptions and marketplace links
- Contact information or link

### tos.html
- Terms of Service covering all published monday Marketplace apps
- Must include: acceptable use, disclaimers, governing law

### privacy.html
- Privacy Policy covering all published monday Marketplace apps
- Must comply with monday Marketplace requirements
- Must include: what data is collected, how it is used, third-party sharing, contact for data requests

## monday Marketplace Requirements
- TOS and Privacy Policy URLs must be stable, publicly accessible HTTPS links
- URLs are submitted during app registration and reviewed by monday
- Pages must be human-readable (not redirects or login-gated)

## Project Rules
- No frameworks, no npm, no build steps — plain HTML/CSS only
- Keep it professional but minimal — this is a compliance and credibility page, not a portfolio showcase
- Legal pages (TOS, privacy) take priority over design

---
> Source: [billykov/billykov-site](https://github.com/billykov/billykov-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

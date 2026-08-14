## michalkomar-com

> - `index.html` is the entire product: terminal UI + command router + man-page profile + inline CSS + JSON-LD. No build step, no external assets besides `favicon.svg`/`social-card.png`.

# Repository Guidelines

## Structure
- `index.html` is the entire product: terminal UI + command router + man-page profile + inline CSS + JSON-LD. No build step, no external assets besides `favicon.svg`/`social-card.png`.
- The man page (`#man`) is the SEO surface — real visible content, never hidden. The terminal is progressive enhancement on top.
- Machine-readable mirrors: `llms.txt`, `about.json`, `ai-profile.json`. Infrastructure: `robots.txt`, `sitemap.xml`, `_headers`.

## Commands & content
- Commands live in the `CMDS` registry + `ALIAS` map inside `index.html`; output templates are `<template data-cmd="...">` blocks — one reflowable variant, no fixed-width ASCII boxes, must wrap cleanly at 320px.
- Legacy aliases (`w`, `ps`, `cat ~/.bashrc`, `apropos`, `finger --linkedin`, …) must keep working.
- Any copy change propagates to: man page, templates, JSON-LD `@graph`, `llms.txt`, `about.json`, `ai-profile.json`, plus `dateModified`/`lastmod` dates.
- Facts must stay verifiable (site history, LinkedIn, Trask/Ackee case studies). The repo is public: never commit personal email addresses or secrets.

## Build, test, deploy
```bash
npm install
npm run dev      # wrangler pages dev . --local true
npm test         # syntax-checks inline JS
npm run deploy   # manual; pushing to main also auto-deploys via Cloudflare Git integration
```

## Style
- Keep the CLI aesthetic: lowercase commands, man-page voice, amber/paper themes via CSS custom properties, system mono stack (no webfonts).
- Accessibility floor: visible `:focus-visible`, `prefers-reduced-motion` respected, input ≥16px, `role="log"` + `aria-live` on output, AA contrast in both themes.

## Commits & PRs
- Imperative, focused commits (`feat: add uptime command`). PRs: note affected commands, manual tests run, SEO/meta updates, and include 375px + desktop screenshots in both themes.

---
> Source: [michalkomar/michalkomar.com](https://github.com/michalkomar/michalkomar.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

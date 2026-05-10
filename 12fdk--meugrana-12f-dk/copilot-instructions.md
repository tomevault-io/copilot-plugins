## meugrana-12f-dk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Landing page for **MeuGrana**, an iOS finance app (parcelas/financas). Static site hosted on GitHub Pages at meugrana.12f.dk.

App Store: https://apps.apple.com/br/app/meugrana-parcelas-finan%C3%A7as/id6759177555

## Development

No build tools, package manager, or dependencies. Pure HTML/CSS/JS.

**Run locally:**
```bash
python3 -m http.server 8000
```
Then open http://localhost:8000

**Deploy:** Push to `main` — GitHub Pages auto-deploys via CNAME (`meugrana.12f.dk`).

## Architecture

```
index.html              # Main landing page
privacy-policy.html     # Privacy policy (bilingual PT/EN)
css/style.css           # All styles (~738 lines, CSS custom properties)
js/main.js              # i18n, scroll animations, mobile nav (~197 lines)
images/screenshots/     # Locale-specific: pt-BR/ and en-US/ (1-5.png)
CNAME                   # GitHub Pages custom domain
```

### Internationalization (i18n)

Client-side language switching implemented in `js/main.js`:
- Translation dictionary maps `data-i18n` attributes to PT-BR/EN-US strings
- Language preference persisted in LocalStorage
- Browser language auto-detected on first visit
- Screenshots swap between `images/screenshots/pt-BR/` and `images/screenshots/en-US/`

When adding translatable text, add `data-i18n="key"` to the HTML element and add both `pt` and `en` entries in the `translations` object in `js/main.js`.

### CSS Design System

Brand colors defined as CSS custom properties in `css/style.css`:
- `--brand-green: #006B4F` (primary)
- Semantic colors: income green, installment blue, warning orange, expense red
- Mobile-first responsive breakpoints: 375px, 768px, 1024px, 1400px

### Scroll Animations

Uses IntersectionObserver for fade-in effects. Respects `prefers-reduced-motion`.

## Testing

No automated tests. Manually verify:
- Both languages (toggle in nav)
- Mobile and desktop viewports
- iOS Safari (primary target audience)

---
> Source: [12fdk/meugrana.12f.dk](https://github.com/12fdk/meugrana.12f.dk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->

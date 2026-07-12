## globetempusplus

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Static marketing landing page for **GlobeTempus+** (also branded as **Tempus+**), a time zone converter iOS app by CarryApps. Deployed to `carryapps.com` via GitHub Pages (CNAME file). No build step — the entire site is a single `index.html` with inline CSS and JS, plus `sw.js` for PWA caching.

## Development

Open directly in a browser — no server or build tool required:
```
open index.html
```

For local serving with proper PWA/service-worker behavior (service workers require a server context):
```
python3 -m http.server 8080
```

To update the PWA cache when assets change, bump the cache name in `sw.js`:
```js
const CACHE = 'tempus-v2';  // increment version here
```
and add/remove URLs from the `ASSETS` array.

## Architecture

**Single-file layout** — all HTML, CSS, and JS live in `index.html`. There are no external stylesheets, frameworks, or JS libraries. The service worker (`sw.js`) is a separate file and caches the key assets listed in `ASSETS`.

**Page sections** (in order):
- `nav` — fixed, blurred backdrop, App Store badge link
- `#top / .hero` — animated gradient background, headline, hero phone image
- `#screens / .showcase` — horizontal scrolling app screenshot carousel
- `#availability` — "Available now" feature callout
- Feature callouts (time picker, edit list) — full-bleed alternating layout sections
- `#features / .grid-section` — feature grid cards
- `.how-section` — "How it works" steps
- `#download / .cta-section` — final CTA with App Store badge
- `footer`

**Design tokens** (CSS custom properties in `:root`):
- `--accent: #FF5722` — orange-red primary accent (note: the user brief says `#FF4500` but the actual code uses `#FF5722`)
- `--bg: #1C1C1E`, `--bg-2: #202022`, `--bg-3: #26262A` — dark background steps
- `--text: #FFFFFF`, `--muted: rgba(235,235,245,0.55)`
- `--section-pad-x: 52px` (collapses to `24px` on mobile)
- `--nav-h: 64px`

**Responsive breakpoints** are defined in `@media` blocks near the bottom of the `<style>` tag. The hero uses `clamp()` for fluid type sizing.

**Images** (`/images/`): all iPhone screenshots follow the naming convention `{version}_{index}_iPhoneX_{descriptor}.png`. `hero-phone.png` is the hero composite.

**App Store badge**: the `Download_on_the_App_Store_Badge_US-UK_RGB_wht_092917.svg` at the project root is also inlined as SVG in the nav. When updating the badge or App Store URL, update both places.

## SEO / PWA Notes

- Meta description and `<title>` are in `<head>` — keep them updated with any branding changes.
- `theme-color` meta is set to `#1C1C1E`.
- The service worker uses a cache-first strategy for listed assets, falling back to network. When adding new images or pages, add them to `ASSETS` in `sw.js`.
- `scroll-padding-top` on `html` accounts for the fixed nav height — do not also set `scroll-margin` on anchor targets (doubles the gap).

---
> Source: [IevgeniiL/GlobeTempusPlus](https://github.com/IevgeniiL/GlobeTempusPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->

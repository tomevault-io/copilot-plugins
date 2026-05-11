## aquafire-portal

> Static documentation and tools portal for **Aquafire** fireplace products (by Lumina Brands). Dark-themed, mobile-responsive site with interactive calculators — no frameworks, no build tools, no runtime dependencies.

# CLAUDE.md — Aquafire Portal

## Project Overview

Static documentation and tools portal for **Aquafire** fireplace products (by Lumina Brands). Dark-themed, mobile-responsive site with interactive calculators — no frameworks, no build tools, no runtime dependencies.

**Stack:** Vanilla HTML5 + CSS3 + JavaScript. Static files served directly from root.

## File Map

### Pages

| File | Purpose |
|------|---------|
| `index.html` | Landing hub — bento grid nav, model cards, section links |
| `aquafire-pro.html` | Comprehensive Pro model guide (specs, wiring, troubleshooting, video guide) — **largest page (~1,400 lines)** |
| `enclosure-guide.html` | Interactive enclosure dimension calculator with 3D isometric diagram |
| `water-care.html` | Water hardness lookup (ZIP code DB) and softener replacement calculator |
| `quick-start.html` | Model selection page linking to individual guides |
| `getting-started.html` | Placeholder — "coming soon" |
| `support.html` | Support hub with stub cards (warranty, claims, troubleshooting, FAQs) |

### Stylesheets

| File | Scope |
|------|-------|
| `hub.css` | **Shared** — site nav, footer, page headers, bento tiles, common components |
| `styles.css` | Enclosure guide — forms, SVG diagram, cards, step layout |
| `water-care-styles.css` | Water care — hardness scale, map tiles, calculator UI |

### JavaScript

| File | Scope |
|------|-------|
| `app.js` | Enclosure guide — model data, dimension math, SVG/Canvas rendering, slider controls |
| `water-care-app.js` | Water care — 2,000+ ZIP code hardness DB, autocomplete, US map, replacement timeline |
| `embed.js` | Strips nav/footer when page loaded in iframe (`?embed` query param) |

## Architecture

```
Root (flat — no subdirectories)
├── Shared: hub.css, embed.js
├── Landing: index.html
├── Product Guide: aquafire-pro.html (self-contained CSS/JS inline)
├── Enclosure Tool: enclosure-guide.html + styles.css + app.js
├── Water Care Tool: water-care.html + water-care-styles.css + water-care-app.js
└── Stubs: quick-start.html, getting-started.html, support.html
```

No build process. No package.json. No bundler. Edit files directly and deploy.

## Design System

### Colors (CSS custom properties in hub.css)

```
Backgrounds:   --bg: #121417 | --surface: #1b1e24 | --surface-alt: #22262e
Borders:        --border: #2c3038 | --border-warm: #3a3229
Text:           --text: #e4e5e9 | --text-muted: #878c99
Red (primary):  --red: #c0392b | --red-dark: #a93226 | --red-glow: rgba(192,57,43,0.15)
Amber:          --amber: #e8a838 | --amber-dim: #c78820
Blue:           --blue: #4da6e8 | --blue-dim: #2980b9
Ember:          --ember: #d45a20
Radius:         --radius: 14px | --radius-sm: 8px
```

### Typography

- **Headings:** Poppins (600, 700) via Google Fonts
- **Body:** Inter (400, 500, 600, 700) via Google Fonts

### Component Patterns

- **Bento tiles** (`.bento-tile`) — colored accent bar, hover glow, CTA arrow
- **Cards** (`.card`, `.hub-card`, `.model-card`, `.support-card`) — surface bg, border, radius
- **Site nav** (`.site-nav`) — sticky, backdrop-blur, hamburger on mobile
- **Buttons** (`.btn-primary`) — red with glow shadow
- **Accordions** — in aquafire-pro.html troubleshooting section (toggle via JS)

### Responsive Breakpoints

- `800px` — nav collapses to hamburger
- `700px` — grid shifts (2-col → 1-col)
- `600px` — compact mobile layout

## External Dependencies

- **Google Fonts** — Inter + Poppins (preconnected)
- **Shopify CDN** — product images and logo (`cdn.shopify.com/s/files/1/0671/5562/4256/`)
- **No JS libraries.** No analytics. No polyfills.

## Navigation Structure

All pages share the same nav bar (defined inline in each HTML file):

```
Home → Getting Started → Quick Start → Enclosure Guide → Water Care → Support
```

Footer links also appear on index.html and aquafire-pro.html.

## Key Conventions

- **No build step.** All changes are live immediately in source files.
- **Dark theme only.** No light mode toggle — everything uses the dark palette.
- **Inline styles in aquafire-pro.html.** That page has its own `<style>` block (~400 lines) since it was built as a self-contained guide. The other tools use separate CSS files.
- **Nav is duplicated** across all HTML files (no templating). When changing nav links, update every page.
- **Model data lives in `app.js`** as the `MODELS` object — Original, Pro, Lite each with 3 sizes (18", 24", 30"/36"). Update there for spec changes.
- **Water hardness DB is in `water-care-app.js`** — `WATER_HARDNESS_DB` array of `[zip_prefix, city, state, ppm]` tuples.
- **Embed mode:** Append `?embed` to any page URL to hide nav/footer (for Shopify iframe embedding).
- **SVG diagrams** are generated in JS via string concatenation (app.js `drawCutoutDiagram` and isometric renderer).
- **Fractions** are displayed as proper fractions (e.g., 14 1/8") via `toFrac()` in app.js.

## Development History

This portal evolved through iterative Claude Code sessions:

1. **Foundation** — Enclosure dimension calculator with SVG cutout diagrams, later restyled with Aquafire red branding
2. **Water care tool** — ZIP code hardness database, US map visualization, replacement timeline calculator
3. **Hub & navigation** — Bento grid homepage, model cards, shared nav/footer across all pages
4. **Aquafire Pro guide** — Comprehensive product page with specs, video guide, troubleshooting accordions, hub dashboard redesign
5. **Embed support** — iframe-friendly mode for Shopify integration
6. **Product images** — Shopify CDN integration for model cards and accessories
7. **AR Cutout Visualizer** — Camera overlay tool (built → iterated → removed; browser-based AR without depth sensing was unreliable)

## Gotchas

- **Nav duplication:** There's no shared template. Changing navigation means editing 7+ HTML files.
- **aquafire-pro.html is large** (~1,400 lines with inline CSS/JS). Read specific sections rather than the whole file.
- **styles.css is enclosure-specific** despite the generic name. Shared styles are in `hub.css`.
- **No local dev server configured.** Open files directly or use any static server (`python -m http.server`, etc.).
- **Images are all on Shopify CDN** — no local image assets in the repo.

---
> Source: [LuminaBrands/aquafire-portal](https://github.com/LuminaBrands/aquafire-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->

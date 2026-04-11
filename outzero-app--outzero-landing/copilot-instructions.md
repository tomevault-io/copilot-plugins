## outzero-landing

> Instructions for AI coding agents working on the Outzero landing page (Astro + Tailwind CSS v4).

# AGENTS.md — Outzero Landing Page

Instructions for AI coding agents working on the Outzero landing page (Astro + Tailwind CSS v4).

## Project Overview

Static landing page for [outzero.app](https://outzero.app). Built with **Astro 6** (zero-JS static output) and **Tailwind CSS v4** (via `@tailwindcss/vite`). Deployed via **GitHub Pages**.

## Build / Dev Commands

```bash
npm run dev        # Start dev server (localhost:4321)
npm run build      # Production build → dist/
npm run preview    # Preview production build locally
```

## Project Structure

```
outzero-landing/
├── public/
│   ├── fonts/              # Poppins TTF (Light, SemiBold, Bold)
│   ├── icons/spots/        # Spot type SVG icons (from Flutter app)
│   ├── images/             # Logos (SVG/PNG), icon mark
│   ├── favicon.png         # Browser tab icon
│   └── og-image.png        # Open Graph preview (1200x630)
├── src/
│   ├── components/         # Astro components (Navbar, Hero, etc.)
│   ├── layouts/
│   │   └── Layout.astro    # Base HTML layout (head, meta, fonts)
│   ├── pages/
│   │   └── index.astro     # Landing page entry point
│   └── styles/
│       └── global.css      # Design system (CSS vars + Tailwind config)
├── astro.config.mjs        # Astro configuration
├── package.json
└── AGENTS.md               # This file
```

## Design System

### Source of Truth

The design system is defined in `src/styles/global.css` and mirrors `lib/src/theme/app_theme.dart` from the Flutter app. **NEVER hardcode colors, fonts or spacing**.

### Colors — CSS Custom Properties

All colors are exposed as CSS custom properties on `:root` (dark theme) and `[data-theme="light"]` (light theme).

| Token | Dark | Light | Usage |
|-------|------|-------|-------|
| `--background` | `#000000` | `#FFFFFF` | Page background |
| `--surface` | `#424242` | `#B3B4B5` | Cards, elevated UI |
| `--text-primary` | `#FFFFFF` | `#000000` | Main text |
| `--text-secondary` | `#B3B4B5` | `#424242` | Secondary text |
| `--tag-green` | `#CDFAB1` | `#2B4F0A` | Brand accent, CTAs |
| `--tag-green-fg` | `#000000` | `#FFFFFF` | Text ON tag-green |
| `--border` | `#424242` | `#424242` | Borders, dividers |
| `--button-bg` | `#424242` | `#73ABBF` | Secondary buttons |
| `--button-text` | `#FFFFFF` | `#FFFFFF` | Button text |

Additional tokens: `--error`, `--success`, `--warning`, `--pending`, `--social-button`, `--admin-action`.

Raw palette tokens (for gradients/effects only): `--color-brand-green`, `--color-accent-blue`, etc.

### Typography

- **Font**: Poppins (bundled in `/public/fonts/`)
  - `font-weight: 300` — body, paragraphs (Poppins Light)
  - `font-weight: 600` — headings, buttons, labels (Poppins SemiBold)
  - `font-weight: 700` — strong emphasis (Poppins Bold)

### Border Radius

Consistent `8px` — use `rounded-[var(--radius-brand)]` or `var(--radius-brand)` in inline styles.

### Using Colors in Templates

```html
<!-- Tailwind arbitrary values (preferred for bg/text) -->
<div class="bg-[var(--surface)] text-[var(--text-primary)]">

<!-- Inline style (fallback when Tailwind syntax is verbose) -->
<span style="color: var(--tag-green);">Highlighted</span>
```

### Spot Type Icons

SVG icons at `/icons/spots/{name}.svg`. They have `fill="#fff"` baked in. Use the `.spot-icon` CSS class which applies `filter: brightness(0) invert(1)` (dark) / `filter: brightness(0)` (light) to adapt them.

### Theme Switching

The landing defaults to dark theme (`data-theme="dark"` on `<html>`). To support light theme, change the attribute to `data-theme="light"`. All semantic tokens and `.spot-icon` / `.logo-img` / `.footer-logo` / `.hero-icon` classes auto-adapt.

## Code Style

- **Components**: One component per `.astro` file in `src/components/`
- **No frameworks**: Pure Astro components (zero JS shipped by default)
- **Minimal JS**: Only for scroll-based effects (navbar backdrop). Inline `<script>` tags.
- **Responsive**: Mobile-first. Breakpoints: `sm:` (640), `md:` (768), `lg:` (1024), `xl:` (1280)
- **Accessibility**: Semantic HTML, `alt` attributes, `aria-label` on icon-only links, `aria-hidden` on decorative elements.
- **Images**: Use `loading="lazy"` for below-the-fold images.

## Assets Origin

All assets are copied from the Flutter app (`outzero_app`):

| Asset | Source |
|-------|--------|
| Poppins fonts | `assets/fonts/Poppins-*.ttf` |
| SVG logos | `assets/images/logos/outzero_logo_*.svg` |
| SVG icon mark | `assets/images/icons/outzero_icon_*.svg` |
| Spot type icons | `assets/icons/spots/*.svg` |
| Favicon | `web/favicon.png` |
| OG image | `web/og-image.png` |

When updating assets, re-copy from the Flutter project source.

## Deployment

Hosted via **GitHub Pages** from the `outzero-app` GitHub organization. Custom domains: `outzero.app` and `www.outzero.app`.

## Quick Checklist

- [ ] Never hardcode colors — use CSS custom properties
- [ ] Use `font-weight: 300` for body, `600` for headings/buttons
- [ ] Use `rounded-[var(--radius-brand)]` for border radius
- [ ] Use `.spot-icon` class on spot type SVG `<img>` tags
- [ ] Add `alt` text to all meaningful images
- [ ] Test responsiveness at 320px, 768px, 1024px, 1440px
- [ ] Test both dark and light themes
- [ ] Run `npm run build` before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/outzero-app)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/outzero-app)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

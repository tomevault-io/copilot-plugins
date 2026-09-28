## tepate-web

> > Keep this file concise, current, and action-oriented. Link to repo docs instead of repeating them.

# AGENTS.md — Ingeniería Vial TEPATE

> Keep this file concise, current, and action-oriented. Link to repo docs instead of repeating them.

---

## Project Overview

This is the corporate website for **Ingeniería Vial TEPATE, S.A. de C.V.**. It is a static multi-page Astro site for a Mexican B2B road-safety manufacturer/installer. All user-facing content is in Spanish (Mexico), and the visual system is intentionally industrial/brutalist: black surfaces, neon accents, square corners, and monospace details.

The site is not a SPA. Navigation between pages is a full request. Keep changes aligned with the Astro static architecture and avoid client-side routing.

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework / build tool | Astro | 5.x |
| Bundler | Vite | via Astro |
| CSS framework | Tailwind CSS | 4.1.14 (via `@tailwindcss/vite`) |
| Language | TypeScript | 5.7.3 |
| UI islands | React | 18.x (`ContactForm`, `ProductModal`) |
| Runtime target | ES2022 / ESNext modules | — |
| Package manager | npm | — |
| Node engine | 20.x | — |
| Deployment | Vercel | — |
| Analytics | Vercel Analytics (`@vercel/analytics`) | 2.0.1 |

React is used only for isolated Astro islands. Do not convert the site into a React SPA.

---

## Project Structure

Key files and folders:

- [src/pages/](src/pages) for static routes (`index.astro`, `productos.astro`, `servicios.astro`, etc.)
- [src/layouts/Layout.astro](src/layouts/Layout.astro) for shared head/chrome/scripts
- [src/components/](src/components) for Astro components and React islands
- [src/scripts/](src/scripts) for isolated DOM modules
- [src/styles/global.css](src/styles/global.css) as the stylesheet entry point; it imports modular styles from [src/styles/](src/styles)
- [public/](public) for static assets served from `/`
- [vercel.json](vercel.json) for edge headers, CSP, and cache rules
- [AUDIT_REPORT.md](AUDIT_REPORT.md) for remediation history and validation notes
- [FIX_ISSUES_PROMPT.md](FIX_ISSUES_PROMPT.md) for the original fix scope and verification checklist

---

## Build and Development Commands

```bash
# Install dependencies
npm install

# Start development server (port 3000, host 0.0.0.0)
npm run dev

# Production build (outputs to dist/)
npm run build

# Preview production build locally
npm run preview

# Astro + TypeScript check
npm run check

# TypeScript-only check
npm run lint

# Clean build output
npm run clean
```

Important:
- There is no automated test suite in this repo.
- The main code-quality gate is `npm run check` (`astro check && tsc --noEmit`).
- `npm run clean` uses `rm -rf`; on Windows, prefer deleting `dist/` manually if needed.

---

## Build Configuration

### Astro (`astro.config.mjs`)

- **Static output:** `output: 'static'` with file-style HTML output.
- **Routes:** Pages are generated from `src/pages/*.astro`.
- **Sitemap:** `@astrojs/sitemap` generates sitemap output during build.
- **Path alias:** `@/` maps to the project root.
- **Tailwind CSS:** Integrated through Vite with `@tailwindcss/vite`.

### TypeScript (`tsconfig.json`)

- `target`: ES2022
- `module`: ESNext
- `moduleResolution`: bundler
- `isolatedModules`: true
- `noEmit`: true
- `allowImportingTsExtensions`: true
- `paths`: `@/*` → `./*`

---

### Code Organization

### Entry Points

- `src/layouts/Layout.astro` owns the shared document shell, metadata, header/footer slots, Vercel Analytics and rich-motion bootstrap.
- `src/pages/*.astro` owns route content.
- React islands are reserved for stateful widgets such as the quote form and product modal.

### Script Modules / Islands

| Module | Responsibility |
|--------|---------------|
| `analytics.ts` | One-liner wrapper around `@vercel/analytics` `inject()`. |
| `animations.ts` | Optional rich-motion layer. It is skipped for reduced motion, coarse pointers and `/contacto`. |
| `ContactForm.tsx` | B2B quote form. Reads URL intent, validates fields, posts to `PUBLIC_FORMSPREE_ENDPOINT`, and offers WhatsApp/email fallback when not configured. |
| `ProductModal.tsx` | Product detail modal with focus return, Escape close, basic focus loop and quote/WhatsApp CTAs. |
| Inline Astro scripts | Small page-local behavior such as mobile menu, accordions, filters, lightbox and sticky CTA. |

### Styling

The stylesheet is modular now. [src/styles/global.css](src/styles/global.css) imports the layered files under [src/styles/](src/styles); do not reintroduce large inline CSS blocks unless there is a strong reason.

| File | Responsibility |
|------|---------------|
| [src/styles/base.css](src/styles/base.css) | Tokens, reset, focus states, motion reduction, print styles |
| [src/styles/layout.css](src/styles/layout.css) | Shared chrome: topbar, nav, mobile menu, footer, WhatsApp float, back-to-top |
| [src/styles/components.css](src/styles/components.css) | Buttons, cards, forms, accordion, pagination, modal, filters |
| [src/styles/home.css](src/styles/home.css) | Homepage sections |
| [src/styles/pages.css](src/styles/pages.css) | Secondary page layouts and content patterns |

**Design system tokens (excerpt):**
```css
--neon: #E1FF00;        /* Primary accent */
--black: #000000;       /* Page background */
--asphalt: #0A0A0A;     /* Elevated surfaces */
--coal: #0F0F0F;        /* Cards */
--char: #141414;        /* Hover cards */
--steel: #262626;       /* Borders */
--bone: #CCCCCC;        /* Body text */
--white: #F2F2F2;       /* Headlines */
--fd: 'Barlow Condensed', sans-serif;  /* Display font */
--fb: 'Barlow', sans-serif;            /* Body font */
--fm: 'IBM Plex Mono', monospace;      /* Mono font */
```

---

## Page Architecture

Each page is an Astro route in [src/pages](src/pages). They share:

- The same `<nav>` structure (with dropdowns and mobile menu)
- The same footer structure
- The same [src/layouts/Layout.astro](src/layouts/Layout.astro) shell
- The same [src/styles/global.css](src/styles/global.css) stylesheet
- Google Fonts loaded from `fonts.googleapis.com` (Barlow Condensed, Barlow, IBM Plex Mono)

Pages include SEO metadata per page:
- Unique `<title>` and `<meta name="description">`
- Open Graph and Twitter Card tags
- Canonical URL
- JSON-LD structured data (`LocalBusiness`, `ItemList`, etc.)

---

## Development Conventions

### Comment Markers
Preserve existing HTML comment markers such as `MOD-*` and `AUDIT` unless you are explicitly refactoring the affected section. They are part of the repo’s editing history.

### CSS Conventions
- Class names are lowercase, hyphenated, and often abbreviated (`.hprod`, `.svc`, `.gitem`, `.pcat`, `.mvv`).
- BEM-like naming is not used.
- Keep media queries near the rules they affect.
- `!important` is reserved for global resets or other unavoidable overrides.

### Accessibility (A11y) Requirements
New interactive UI must follow the existing patterns:
- Use real `<button>` elements for triggers.
- Keep `aria-expanded`, `aria-controls`, and Escape handling in sync for expandable widgets.
- Trap focus in modals and return focus to the opener.
- Use `aria-invalid` and `aria-describedby` for form errors.
- Keep visible `:focus-visible` outlines and `prefers-reduced-motion` behavior intact.
- Preserve the skip link and `<main id="main-content">` structure on every page.

### Image Handling
- Hero/gallery images live under `/TEPATE_Imagenes_Organizadas/`.
- Brand logos live under `/TEPATE_Marcas/`.
- Use `loading="lazy"` for non-critical images.
- Use `fetchpriority="high"` and `decoding="async"` for hero images.
- Keep CLS protections such as `aspect-ratio` in place.

---

## Deployment

### Vercel (`vercel.json`)

The deployment configuration includes:

- **Security headers:**
  - `Content-Security-Policy` (CSP) — configured for current dependencies (Google Fonts, Formspree, Vercel scripts)
  - `X-Frame-Options: SAMEORIGIN`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` (restricts camera, microphone, geolocation)
  - `Strict-Transport-Security` (HSTS)
- **Cache headers:**
  - CSS/JS/woff: `max-age=31536000, immutable`
  - Images: `max-age=15552000`
  - PDFs: `max-age=86400` + `Content-Disposition: attachment`
- **Redirects:** `/index.html` → `/` (permanent)
- **Clean URLs:** Disabled (`.html` extensions are preserved in URLs)

Warning: if you add a new external domain, update the CSP in [vercel.json](vercel.json) or production will block it.

### Environment Variables

The `.env.example` file documents two variables used when the app runs inside **Google AI Studio**:

| Variable | Purpose |
|----------|---------|
| `GEMINI_API_KEY` | Injected by AI Studio for Gemini API calls |
| `APP_URL` | Injected by AI Studio with the Cloud Run service URL |

For normal local development, these are not required.

---

## Testing Strategy

There is no automated test suite in this repo. Validate changes with:
1. `npm run lint`
2. `npm run build`
3. Manual keyboard checks for nav, modal, mobile menu, and contact form behavior

If tests are added later, keep them lightweight and focused on the shared navigation/modal/form flows.

---

## Security Considerations

1. CSP and related headers are enforced through [vercel.json](vercel.json).
2. The contact form uses client-side `fetch` to Formspree; preserve the current validation and ARIA semantics if you change it.
3. Do not commit secrets. The `.env.example` file only documents AI Studio variables.
4. Vercel Analytics is already accounted for in the CSP.

---

## Useful References

- [README.md](README.md) for local run instructions and the short project overview
- [AUDIT_REPORT.md](AUDIT_REPORT.md) for implementation history and validation notes
- [FIX_ISSUES_PROMPT.md](FIX_ISSUES_PROMPT.md) for the remediation scope and checklist
- [vercel.json](vercel.json) for edge headers and caching rules
- [sitemap.xml](sitemap.xml) for the indexed page list

---

## Common Pitfalls

1. Do not treat this as a SPA.
2. Do not add a frontend framework unless the user explicitly asks.
3. Do not bypass [src/styles/global.css](src/styles/global.css); route styles through the modular `src/styles/` files.
4. Do not use `app/applet/optimize.js` as part of the normal build; it is a legacy HTML rewriting script.

---
> Source: [EJMM17/tepate-web](https://github.com/EJMM17/tepate-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->

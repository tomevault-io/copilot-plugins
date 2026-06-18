## 2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

- `npm run dev` — Start Vite dev server (defaults to http://localhost:5173)
- `npm run build` — TypeScript check + Vite production build (outputs to `dist/`)
- `npm run preview` — Preview the production build locally

No test runner or linter is configured.

## Deployment

Pushes to `main` trigger `.github/workflows/deploy.yml`, which runs `npm ci && npm run build` and deploys `dist/` to GitHub Pages. The custom domain is `priyamvadaux.com` (configured via `public/CNAME`).

The Vite base path is `/portfolio/` in production mode (for GitHub Pages) and `/` in development — see `vite.config.ts`.

## Architecture

This is a personal portfolio site for a Lead Product Designer. It's a React 18 + TypeScript SPA built with Vite, styled with Tailwind CSS 3 plus extensive custom CSS.

### Routing

Uses `react-router-dom` v7 with `HashRouter` (imported as `BrowserRouter` in `App.tsx`). Two routes:
- `/` → `Home` page (single-page scrolling portfolio)
- `/case-study/raqam` → `CaseStudyRaqam` page (detailed case study)

### Page Structure

**Home** (`src/pages/Home.tsx`) composes these section components in order: `Cursor` → `Navigation` → `Hero` → `Ticker` → `Work` → `Process` → `About` → `Contact` → `Footer`. It sets up a global IntersectionObserver for `.rv` (reveal) elements.

**CaseStudyRaqam** (`src/pages/CaseStudyRaqam.tsx`) is a self-contained ~42KB file with all case study content inline.

### Styling Approach

- **Tailwind** for utility classes, configured in `tailwind.config.ts` with a custom dark color palette (black/s1/s2/s3 surfaces, gold accent, cream text)
- **CSS custom properties** defined in `src/index.css` (`--black`, `--gold`, `--white`, `--muted`, `--dim`, `--border`, `--pad`, etc.)
- **Inline styles** via `React.CSSProperties` objects (most components define a `const S: Record<string, React.CSSProperties>` at the top)
- **CSS classes** in `index.css` for the bento grid layout (`.bento`, `.b-raqam`, etc.), card system (`.card`, `.cl`, `.cn`, `.ca`, `.ci`), and reveal animations (`.rv`, `.on`)
- Custom cursor is hidden (`cursor: none` on body) and replaced by a gold dot + ring (`Cursor.tsx`)

### Design Tokens

Two font families: `Cormorant Garamond` (serif, headings) and `Outfit` (sans, body). Loaded from Google Fonts in `index.html`.

Color palette: dark surfaces (`#08080f` through `#181828`), gold accent (`#C9A84C`), cream text (`#f0ede6`).

### Key Patterns

- **Reveal-on-scroll**: Elements with class `rv` are observed by IntersectionObserver; when visible, class `on` is added to trigger CSS transitions. Stagger delays use `.d1`–`.d5` classes.
- **Bento grid**: The Work section uses a 12-column CSS Grid with named placement classes for a magazine-style layout. Responsive breakpoints at 1200px (6-col) and 768px (1-col).
- **Static images** go in `public/images/`.

---
> Source: [priyauxd/2026](https://github.com/priyauxd/2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

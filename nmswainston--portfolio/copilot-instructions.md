## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start Vite dev server
npm run build        # Production build → dist/
npm run preview      # Preview production build locally
npm run lint         # Run ESLint
npm run screenshots  # Capture project screenshots via Puppeteer (scripts/capture-screenshots.js)
npm run push         # Run scripts/push.ps1 (PowerShell deploy helper)
```

No test suite is configured.

## Architecture

**Stack**: React 19 + Vite + Tailwind CSS v4 + React Router v7. Deployed to Netlify with a catch-all SPA redirect (`netlify.toml`).

**Tailwind v4** is loaded via `@tailwindcss/vite` (not the traditional PostCSS plugin). The `vite.config.js` imports and registers it directly — there is no `tailwind.config.js`.

### Routing

Three routes defined in `App.jsx`:
- `/` → `HomePage` (renders `WorkShowcase`, `HowIWork`, `SkillsFocus`, `About`, `WhatLookingFor`, `Contact` in sequence)
- `/work` → `AllWork`
- `/work/:slug` → `ProjectDetail`

### Project data

**`src/sections/projectsData.js`** is the single source of truth for all projects. Each entry has:
- `slug` — URL key for `/work/:slug`
- `featured: true/false` — controls whether it appears on the homepage (`WorkShowcase` shows the first 6 `featured` projects in order)
- `goal`, `process`, `result` — rendered as case study sections in `ProjectDetail`
- `liveUrl` — external live link shown on cards
- `link` — legacy alias for `liveUrl`, kept for backward compatibility
- `image` — imported asset (must be `.png` with a single extension; Vite import requirement noted in the file)

To add a project, add an entry to `projectsData.js` and drop the screenshot in `src/assets/projects/`.

### Styling conventions

`src/index.css` defines the global design system:

- **Theme**: two themes (`light` / `dark`) via `data-theme` attribute on `:root`. CSS variables `--bg`, `--fg`, `--card`, `--line`, `--accent`, `--muted`, `--shadow` are redefined per theme. `ThemeToggle.jsx` handles toggling.
- **Utility classes** in `index.css`: `.container`, `.card`, `.btn`, `.btn-ghost`, `.tag`, `.hint`, `.copy`, `.link-text`, `.work-tile`, `.section`, `.layout-grid`, `.case-study-*` (full case study layout). Use these alongside Tailwind utilities.
- **Section spacing**: `.section` + `.section + .section` margin system using `--section-gap` / `--section-gap-mobile` vars — don't override with ad-hoc margins.
- Tailwind CSS variables are referenced with `(--var)` syntax in JSX (e.g., `bg-(--accent)`), not with `var(--var)`.

### Component roles

- `CaseStudyLayout.jsx` — wraps case study pages; handles hero, prev/next navigation, back link, and live URL CTA. Pass the full `project` object.
- `CaseStudySection.jsx` — renders a single titled section (`goal`/`process`/`result`) within a case study.
- `DeviceMockup.jsx` — renders a laptop or phone frame around a project screenshot. Takes `type`, `image`, `alt`.
- `Card.jsx` — generic card with optional `title` and `actions`.
- `Section.jsx` — thin wrapper that applies `.section .container` classes.

---
> Source: [nmswainston/portfolio](https://github.com/nmswainston/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->

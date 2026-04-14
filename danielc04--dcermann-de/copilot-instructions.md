## dcermann-de

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (exposed on all interfaces)
npm run build      # Type-check, build, then run deploy.sh
npm run lint       # ESLint with zero warnings tolerance
npm run test       # Run tests once (Vitest)
npm run test:watch # Run tests in watch mode
npm run preview    # Preview production build
```

To run a single test file:
```bash
npx vitest run src/__tests__/mergeData.test.ts
```

## Architecture

**Stack:** React 19 + TypeScript + Vite, SCSS (no CSS modules), Bootstrap 5 + react-bootstrap, react-router-dom v7 (single route `/`), i18next for EN/DE localization.

### Data flow

Content is split across three JSON files in `src/assets/text/`:
- `shared_data.json` — language-agnostic metadata (image paths, URLs, dates, icon classes, proficiency numbers)
- `data_english.json` / `data_german.json` — translatable text (titles, descriptions, section names)

At runtime, `mergeData.ts` zips these by index to produce the `LangSpecificData` type consumed by all section components. `App.tsx` reads the current i18n bundle via `i18n.getResourceBundle()` (not `t()`), calls `mergeData`, and passes the result as props down to each section. **Adding or reordering entries requires keeping all three JSON files in sync.**

The `{{AGE}}` placeholder in the description is replaced in `About.tsx` using `calculateAge()` from `src/components/utils/utils.ts`, not by i18next interpolation.

### Component structure

```
src/
  App.tsx                        # Root: loads data, lazy-imports all sections
  i18n.ts                        # i18next init; language persisted in localStorage
  mergeData.ts                   # Merges shared + lang JSON by index
  components/
    sections/                    # One component per page section (Home, About, Projects, Skills, Experience, Contact)
    utils/                       # Sidebar, Footer, LanguageSwitch, ColorThemeSwitch, Loader(Page), Tooltip
  Backgrounds/
    MatrixRain/                  # Canvas-based matrix rain animation (Home hero)
    threejs/                     # Unused three.js background
    svg/                         # SVG grid background overlay
  customHooks/
    useIsInView.ts               # Scroll-based viewport detection (used by MatrixRain to pause when off-screen)
  scss/themes/
    theme-dark.scss              # CSS custom properties for dark theme (data-theme="dark" on body)
    theme-light.scss             # CSS custom properties for light theme
```

All section components receive their data via props from `App.tsx`. They do not fetch data themselves.

### Theming

Color theme is toggled via `data-theme` attribute on `<body>`. CSS variables (`--background-color`, `--text-color`, `--contrast-color`, `--background-color-1/2`) are defined in `src/scss/themes/`. The MatrixRain reads `data-theme` directly to switch between green (dark) and gradient (light) colors.

### Navigation

`Sidebar.tsx` renders hash-links to each section ID. Active section is tracked by finding the section element with the smallest `getBoundingClientRect().top` on scroll. Section IDs must match the keys of `SectionName` in `loaded_data_types.d.ts`.

### Contact form

Uses EmailJS. Credentials (`PUBLIC_KEY`, `SERVICE_ID`, `TEMPLATE_ID`) are in `src/components/sections/EmailjsLogin.ts` — this file is gitignored and must be created locally.

### Tests

Vitest with jsdom. Tests live in `src/__tests__/`. Currently covers `mergeData` and i18n initialization.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DanielC04)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DanielC04)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

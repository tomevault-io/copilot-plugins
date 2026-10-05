## personal-page

> Astro 5 personal site (TypeScript strict, Tailwind v4, MDX). Spanish is the default locale; English lives under `/en/`. Deployed to Cloudflare Pages via GitHub Actions.

# AGENTS.md

Astro 5 personal site (TypeScript strict, Tailwind v4, MDX). Spanish is the default locale; English lives under `/en/`. Deployed to Cloudflare Pages via GitHub Actions.

## Commands

All commands run from the repo root.

- `npm run dev` — Astro dev server at `http://localhost:4321`.
- `npm run build` — static build to `./dist`.
- `npm run preview` — serve the production build locally.
- `npm run check` — `astro check` (typecheck + content schema validation). **This is the only verification step** — there are no tests, linter, or formatter configured.

## Architecture

### Routing and i18n

- `astro.config.mjs` sets `i18n.defaultLocale: 'es'` with `prefixDefaultLocale: false`. Spanish routes are at `/`, English routes are at `/en/`. There is **no `/es/` prefix**.
- `src/i18n/utils.ts` is the single source of truth for route mapping and dictionaries. When adding a new page, update `routes` there **and** add the page pair in `src/pages/` and `src/pages/en/`.
- Helpers: `getDictionary(locale)`, `localizedPath(path, locale)`, `routeFor(key, locale)`, `translatePath(path, from, to)`. Use these instead of hardcoding `/en/...`.
- Dictionaries live in `src/i18n/{es,en}.json`. Add new strings to both files.
- `astro.config.mjs` `site` is a placeholder (`personal-page.example.com`) — affects sitemap + canonical URLs.

### Content collections (projects)

- Schema defined in `src/content.config.ts` with Zod. Each project must have **both** `es.md` and `en.md` in `src/content/projects/<slug>/`, sharing the same `slug` and setting `locale: "es"` / `locale: "en"`.
- `date` is coerced from string; `url` / `repo` must be valid URLs if present; `featured` defaults to `false`.
- Schema violations fail `npm run check`.

### Profile data

- `src/data/profile.json` is the source of truth for competences, experience, and contact info (LinkedIn-derived). Edit JSON here, not in components.

### Styling

- Tailwind v4 (not v3): tokens declared in `src/styles/global.css` via the `@theme` directive. No `tailwind.config.*` file.
- Dark mode uses the `[data-theme="dark"]` attribute selector (`@custom-variant dark`). The `theme-toggle.ts` script in `src/scripts/` handles persistence in `localStorage`. Do **not** rely on `prefers-color-scheme`.
- Fonts: `@fontsource-variable/{fraunces,inter,jetbrains-mono}` imported in `global.css`.

### Client scripts

- `src/scripts/*.ts` (theme-toggle, locale-toggle, mobile-menu, reveal, scroll-pagination) are vanilla TS imported via `<script>` blocks in `.astro` components. No framework. Respect `prefers-reduced-motion` (see `global.css`).

## Deployment

- `wrangler.toml` declares Cloudflare **Pages** (`pages_build_output_dir = "dist"`), not Workers. Don't run `wrangler deploy` — it will misconfigure.
- `.github/workflows/deploy.yml` builds on push to `main` and deploys via `cloudflare/pages-action@v1`.
- Required secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`. The workflow also references a hardcoded `projectName: personal-page`.

## Build quirks

- `vite.build.assetsInlineLimit: 0` in `astro.config.mjs` — every asset is emitted as a separate file. Inlining tricks will be discarded.
- `@astrojs/sitemap` is enabled; the sitemap is generated at build from the configured `site` URL.
- Output is fully static (no SSR adapter).

## What doesn't exist

- No test framework, no ESLint, no Prettier, no Biome. Do not invent commands like `npm test` or `npm run lint`. The verification gate is `npm run check`.

---
> Source: [salvador-lax/personal-page](https://github.com/salvador-lax/personal-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->

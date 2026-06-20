## dendyarmanda-github-io

> Project context for Claude Code. Keep this lean — facts, rules, pointers.

# CLAUDE.md

Project context for Claude Code. Keep this lean — facts, rules, pointers.

## What this is

Personal **CV / portfolio** for Dendy Septian Armanda (Banking Software Engineer).
Bilingual, static, zero-cost. Deployed to **both** GitHub Pages + Vercel.

- 🇮🇩 Indonesian (default): `/`  · 🇬🇧 English: `/en/`
- Print/PDF résumé: `/cv` and `/en/cv`
- Live: https://dendyarmanda.github.io · https://profile-eight-inky-80.vercel.app

## Stack

- **Astro 6** (static output, ~zero client JS) + **Tailwind CSS v4**
- **Node ≥ 22.12** required (Astro 6). Pinned: `package.json` `engines.node: 22.x`.
- Self-hosted fonts via **Fontsource** (variable).

## Commands

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # → dist/
npm run preview  # serve the built dist/
```

## Where things live

- `src/data/cv.ts` — **ALL CV content**, single source of truth. Every field is `{ id, en }`.
- `src/i18n/ui.ts` — UI strings (nav, buttons, section kickers) + `useTranslations(lang)`.
- `src/layouts/Base.astro` — html shell, SEO/OG meta, JSON-LD, font imports, reveal observer, particle bg.
- `src/components/` — Hero, About, Experience, Skills, Projects, Contact, Header, Footer, Resume, ParticleNetwork.
- `src/pages/` — `index.astro` (id), `en/index.astro`, `cv.astro`, `en/cv.astro`.
- `scripts/gen-og.mjs` — regenerates `public/og.png` (social card). `scripts/gen-avatar.mjs` — dummy avatar.
- `.github/workflows/deploy.yml` — GitHub Pages deploy (withastro/action).

Detailed run/edit/deploy guide → `README.md`. Project state + pending work → `HANDOFF.md`.

## Hard rules (don't break these)

1. **Tailwind = PostCSS plugin** (`@tailwindcss/postcss` + `postcss.config.mjs`). Do **NOT** switch to `@tailwindcss/vite` — it crashes on Astro 6's rolldown-vite (`Missing field tsconfigPaths`).
2. **Dark-only theme.** No `dark:` utilities, no `@custom-variant dark`. Write dark-native colors directly. Don't reintroduce a light/theme toggle unless asked.
3. **Fontsource family names keep the `Variable` suffix**: `Inter Variable`, `Space Grotesk Variable`, `JetBrains Mono Variable` (see `@theme` in `src/styles/global.css`). Mismatch → silent fallback to system fonts.
4. **New animated sections must use `data-reveal`** — the scroll-reveal is progressive enhancement (hidden only under `html.js`; reduced-motion + no-JS → visible). Don't gate content visibility on JS any other way.
5. **CI Node version**: keep `node-version: 22` in `.github/workflows/deploy.yml` (Astro 6 needs ≥22.12; the action defaults to Node 20).
6. **Canonical = GitHub Pages.** `site` in `astro.config.mjs` is `https://dendyarmanda.github.io`. Vercel is a mirror.
7. Edit content **only** in `src/data/cv.ts` / `src/i18n/ui.ts` — components render from data, don't hardcode copy.
8. After changing name/role/hero tags, **re-run `node scripts/gen-og.mjs`** so the share card stays in sync.

## Conventions

- i18n routing: `astro.config.mjs` `i18n.routing.prefixDefaultLocale: false` → `id` at root, `en` at `/en/`. Pages pass `lang` + `altHref` to components.
- Section components take a `lang` prop and read from `cv.ts` / `ui.ts`.
- Particle background (`ParticleNetwork.astro`) renders only on main pages (via `showChrome`), never on `/cv`. It needs `html` bg opaque + `body` bg transparent (set in `global.css`).
- Contact form posts to **Web3Forms** (key in `Contact.astro` — public by design; spam honeypot included). `mailto:` fallback below it.
- `/cv` print: `@media print` in `global.css` forces résumé text black + strips glass/gradient. Keep `Resume.astro` free of gradient-text/glass.

## Deploy

Push to `main` → **both** redeploy automatically (GitHub Actions for Pages; Vercel git integration). No manual step.

## Library docs

Follow the global rule: fetch library APIs from **Context7** before writing lib code. Never read `node_modules/`.

---
> Source: [dendyarmanda/dendyarmanda.github.io](https://github.com/dendyarmanda/dendyarmanda.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->

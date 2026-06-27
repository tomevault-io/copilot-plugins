## cartershades-dot-com

> Guide for AI agents working in this repo.

# CLAUDE.md

Guide for AI agents working in this repo.

## What this is

Static personal website for cartershades.com, built with Astro 5. No runtime
backend, no client-side framework. Output is plain HTML deployed via GitHub
Pages (the `public/CNAME` file binds the deployed site to the custom domain).

Design system is documented in [DESIGN.md](DESIGN.md) — read that before
making any visual change. The type system is fluid (continuous scaling with
viewport width); never write breakpoint media queries for type sizing.

## Where to make common changes

| Task | File(s) |
| --- | --- |
| Add a page | new file in `src/pages/` + entry in `src/config/site.ts` |
| Rename or reorder nav | `src/config/site.ts` only |
| Change site title / description / URL | `src/config/site.ts` |
| Change site-wide chrome (head, header, footer) | `src/layouts/BaseLayout.astro`, `src/components/Header.astro`, `src/components/Footer.astro` |
| Add a writing post | new `.md` in `src/content/writing/` (schema in `src/content.config.ts`) |
| Change writing frontmatter schema | `src/content.config.ts` |
| Tweak a type step's sizes | `--fs-<name>-min` / `--fs-<name>-max` in `src/styles/tokens.css` |
| Add a new type step | min/max in `src/styles/tokens.css` + rule in `src/styles/typography.css` (see DESIGN.md) |
| Change global GRAD or ROND | `--axis-grad` / `--axis-rond` in `src/styles/tokens.css` |
| Tweak a base color | `--ink` / `--canvas` / `--accent` in `src/styles/tokens.css` (Tier 1) |
| Add a new color role | new `--color-<role>` in Tier 2 of `src/styles/tokens.css` + document in DESIGN.md |
| Tune dark mode | the `@media (prefers-color-scheme: dark)` block in `src/styles/tokens.css` |
| Tweak a spacing step | `--space-<name>-min` / `--space-<name>-max` + the matching `calc()` slope in `src/styles/tokens.css` |
| Add a new spacing step | min/max + fluid `calc()` in `src/styles/tokens.css` + document the use case in DESIGN.md |
| Tune the responsive viewport range | `--vw-min` / `--vw-max` in `src/styles/tokens.css` (affects every fluid token) |
| Change container max-width or gutter | `--container-max` / `--container-gutter` in `src/styles/tokens.css` |
| Tweak a radius value | `--radius-<name>` in `src/styles/tokens.css` |
| Apply page max-width to a section | wrap the section in `<element class="container">` (defined in `global.css`) |

## Conventions

- **Every page wraps in `BaseLayout`.** Pages contain only the page-specific
  content inside `<BaseLayout title="…">…</BaseLayout>`. Do not duplicate
  `<html>` / `<head>` / header / footer in pages.
- **Single source of truth for nav.** Nav links live in `src/config/site.ts`.
  Do not hardcode link lists anywhere else.
- **Components are PascalCase, one per file**, in `src/components/`.
- **Imports use relative paths** (`../config/site`, `../layouts/BaseLayout.astro`).
  No path aliases configured yet — don't add one for a single import site.
- **TypeScript strict** (extends `astro/tsconfigs/strict`). Type page props via
  the `Props` interface pattern shown in `BaseLayout.astro`.

## What not to do

- Do not add a UI framework (React, Vue, Svelte, etc.). The site is intended
  to stay static-first.
- Do not duplicate layout markup across pages.
- Do not hardcode the site URL — read from `site.url` in `src/config/site.ts`.
- Do not hardcode `font-size`, `line-height`, or font axis values — use the
  tokens in `src/styles/tokens.css`. See [DESIGN.md](DESIGN.md).
- Do not hardcode `margin`, `padding`, or `gap` values — use `--space-*`
  tokens. Spacing is fluid; never write media queries to scale it.
- Do not hardcode `border-radius` values — use `--radius-*` tokens.
- Do not write CSS media queries to scale type — the system is fluid.
- Do not put `wght` / `wdth` / `slnt` / `opsz` inside `font-variation-settings`
  (it silently overrides the standard CSS shorthands). Only `GRAD` and `ROND`
  go through `font-variation-settings`.
- Do not reference Tier 1 color primitives (`--ink`, `--canvas`, `--accent`)
  from components. Use Tier 2 semantic tokens (`--color-text`, `--color-bg`,
  `--color-link`, etc.) so themes stay swappable.
- Do not add a styling library (Tailwind, CSS-in-JS, etc.) — plain CSS with
  custom properties is the system.
- Do not add backwards-compatibility shims, feature flags, or speculative
  abstractions. Prefer deleting unused code over preserving it.

## Commands

```sh
npm run dev      # dev server on http://localhost:4321
npm run build    # static build to ./dist
npm run preview  # preview the production build
```

## Deployment

GitHub Pages, custom domain via `public/CNAME` → `www.cartershades.com`.
The site URL is set in `astro.config.mjs` so canonical links and any future
sitemap resolve correctly.

---
> Source: [carter-livefront/cartershades-dot-com](https://github.com/carter-livefront/cartershades-dot-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->

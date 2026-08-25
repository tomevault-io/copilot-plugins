## mladi-pirati

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server at localhost:4321
pnpm build        # Production build
pnpm preview      # Preview production build
pnpm typecheck    # TypeScript type checking via Astro
```

No test runner is configured.

## Architecture

**Astro 6 site** with server-side rendering (Node.js adapter, standalone mode). All pages live in `src/pages/` using file-based routing. The only API endpoint is `src/pages/api/join-request.ts` (POST) — it validates and proxies membership form submissions to an external endpoint configured via `APPLICATIONS_RECEIVER_MEMBERSHIP_ENDPOINT`.

**Key directories:**
- `src/config/site.ts` — single source of truth for all site content (copy, nav items, program items, social links). Edit this first when changing content.
- `src/components/` — Astro components (no React/Vue)
- `src/layouts/BaseLayout.astro` — shared layout with meta, fonts, global CSS
- `src/scripts/` — client-side vanilla TypeScript, split into `home/` and `join/` submodules
- `src/internal/` — shared server+client logic (validation, form helpers)
- `src/styles/global.css` — Tailwind 4 imports + CSS custom properties for theming

**Form flow:** `src/pages/pridruzi-se.astro` (large join form) → `POST /api/join-request` → external receiver. The endpoint handles both JSON and FormData; FormData triggers a redirect back with `?status=` query params. Validation is shared via `src/internal/join-form.ts`.

**Client scripts:** Loaded via `<script>` tags in page files. `init-home-page.ts` orchestrates the home page: canvas ASCII-art hero animation (`ascii-hero.ts`), rotating quotes (`quote-loop.ts`), and scroll-aware nav (`site-nav.ts`).

## Environment Variables

```
SITE_URL                                   # Canonical URL
ALLOW_INDEXING                             # Set true to enable sitemap and indexing
APPLICATIONS_RECEIVER_MEMBERSHIP_ENDPOINT  # External endpoint for membership form submissions
```

See `.env.example` for defaults.

## Conventions

- All UI text is in **Slovenian** — keep it that way.
- Use `date-fns` for any date formatting/manipulation.
- Dark theme with golden accent `#f0a000` — defined as a CSS custom property.
- `src/config/site.ts` uses `satisfies` type patterns for content objects; maintain this.
- Respect `prefers-reduced-motion` in any new animations.

---
> Source: [mladi-pirati/mladi-pirati](https://github.com/mladi-pirati/mladi-pirati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->

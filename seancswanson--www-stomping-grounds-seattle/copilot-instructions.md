## www-stomping-grounds-seattle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Stomping Grounds Seattle — a community website for shuffle dancers in Seattle hosting battles, exchanges, and seshes. Co-founded by @maemae.xo_ and @michal_woz. Instagram: @stompinggrounds.seattle.

## Commands

```bash
npm run dev          # Start local dev server
npm run build        # Production build
npm run preview      # Build + run via Wrangler locally
npm run check        # Full type/build/deploy dry-run validation
npm run deploy       # Deploy to Cloudflare Workers
npm run cf-typegen   # Regenerate Cloudflare Workers types (wrangler types)
```

## Architecture

**Astro 5** static site with the **Cloudflare Workers adapter** (`@astrojs/cloudflare`). Pages are built to `dist/` and deployed via Wrangler.

Key integrations:
- `@astrojs/mdx` — allows `.mdx` files in pages and content collections
- `@astrojs/sitemap` — auto-generates sitemap on build
- Site URL is configured in `astro.config.mjs` (currently placeholder `https://example.com`)

**Content Collections** — blog posts live in `src/content/blog/` as `.md` or `.mdx` files. The collection schema (`src/content.config.ts`) requires `title`, `description`, and `pubDate`; `updatedDate` and `heroImage` are optional. Use `getCollection("blog")` to query posts in pages.

**Global constants** — `src/consts.ts` exports `SITE_TITLE` and `SITE_DESCRIPTION` used across the site (currently still set to Astro template defaults, not yet updated for Stomping Grounds).

**Styling** — `src/styles/global.css` is Bear Blog-based with CSS custom properties for theming. Component styles are scoped inside each `.astro` file. Typography uses the Atkinson font (served from `public/fonts/`).

**`platformProxy`** is enabled in the Cloudflare adapter config, which is required to access Cloudflare-specific bindings during `npm run dev`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seancswanson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seancswanson)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

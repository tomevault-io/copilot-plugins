## kaimingguo-github-io

> Personal blog/website (kaiming.io) built with Astro 5, deployed to both Cloudflare Workers and GitHub Pages.

# AGENTS.md

## Project Overview

Personal blog/website (kaiming.io) built with Astro 5, deployed to both Cloudflare Workers and GitHub Pages.

## Commands

- `npm run dev` — Local dev server (localhost:4321)
- `npm run build` — Production build using Cloudflare adapter
- `npm run preview` — Build + local preview via Wrangler
- `npm run deploy` — Deploy to Cloudflare Workers via Wrangler
- `npm run check` — Full check: astro build + tsc + wrangler dry-run deploy
- `npm run format` — Format with Prettier (4-space indent, double quotes, trailing commas)

## Dual Deployment

- **Cloudflare Workers** (primary): Uses `astro.config.mjs` with `@astrojs/cloudflare` adapter. Site: `https://www.kaiming.io`
- **GitHub Pages** (secondary): Uses `astro.config.github.mjs` (no adapter, static output). Site: `https://kaimingguo.github.io`. Deployed via GitHub Actions on push to `main`.

## Architecture

- **Content**: Blog posts go in `src/content/blog/` as `.md` or `.mdx` files. Schema defined in `src/content.config.ts` — frontmatter requires `title`, `description`, `pubDate`; optional `updatedDate`, `heroImage`.
- **Pages**: File-based routing in `src/pages/`. Blog listing at `/blog/`, individual posts via `[...slug].astro`.
- **Layouts**: `src/layouts/BlogPost.astro` wraps blog content.
- **Components**: `BaseHead.astro` handles `<head>` (SEO, OpenGraph). `Header.astro`/`Footer.astro` for site chrome.
- **Site constants**: `src/consts.ts` (`SITE_TITLE`, `SITE_DESCRIPTION`).
- **Cloudflare config**: `wrangler.json` — nodejs_compat flag enabled, observability on.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaimingguo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaimingguo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

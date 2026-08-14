## youtube-checker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # start dev server at localhost:3000
npm run build    # production build
npm run start    # serve production build
npm run lint     # ESLint via next lint
```

No test suite configured.

## Environment

`YOUTUBE_API_KEY` — YouTube Data API v3 key, passed server-side from `page.js` to `YouTubeChannelChecker`. Must be restricted in Google Cloud Console to `chennelchecker.online`. Not exposed via `.env.local` yet; set it there for local dev.

## Architecture

**Next.js 15 App Router**, React 19, Tailwind CSS, MDX for blog posts. No database. Deployed at `https://chennelchecker.online`.

### Core feature: YouTube Channel Checker

- `src/app/page.js` — server component; passes `process.env.YOUTUBE_API_KEY` as a prop to `LandingPage`
- `src/components/LandingPage.js` — `"use client"` component; calls YouTube Data API v3 `search` endpoint directly from the browser using the API key received as a prop
- `src/components/ChannelChecker.js` — pure UI component; given search results + query, does fuzzy normalization to determine if an exact match exists (available vs. taken)

The API key flows: env var → server component prop → client component → browser fetch. This is intentional but means the key is visible in client JS; Google Cloud domain restriction is the security boundary.

### Blog system

Static, no CMS. Two parts must stay in sync:

1. `src/lib/data.js` — array of blog metadata objects (`slug`, `title`, `description`, `readTime`, `date`)
2. `src/app/blog/posts/*.mdx` — MDX files, one per slug
3. `src/app/blog/[slug]/page.jsx` — static imports all MDX files and maps slugs to components in `mdxMap`

**Adding a blog post requires updating all three**: add to `data.js`, create the `.mdx` file, add a static import + entry in `mdxMap` in `[slug]/page.jsx`. Dynamic imports are not used; everything is statically bundled.

### Static pages

Landing page content articles live as standalone page routes:
- `src/app/how-to-choose-perfect-youtube-channel-name/page.js` + `src/components/choose-perfect-youtube-chanel.js`
- `src/app/5-mistakes-naming-youtube-channel/page.js` + `src/components/mistakes-while-creating-channel.js`

### MDX configuration

`next.config.mjs` uses `@next/mdx` with `remark-frontmatter` + `remark-mdx-frontmatter`. `mdxRs: false` is required to support these remark plugins. Page extensions include `.mdx`.

### SEO / Analytics

- Global metadata in `src/app/layout.js`
- Sitemap generated at `src/app/sitemap.js` (static, must be manually updated when routes are added)
- Google Analytics via `@next/third-parties` (`gaId: G-Y2M8F4SPBR`)
- Google AdSense meta tag in `<head>` (`ca-pub-1936231329564538`)
- Canonical URL hardcoded to `https://chennelchecker.online` in layout

---
> Source: [tejasg910/youtube-checker](https://github.com/tejasg910/youtube-checker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

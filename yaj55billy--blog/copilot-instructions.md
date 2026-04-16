## blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server (host: true, so accessible on LAN)
pnpm build        # Build production site to ./dist/
pnpm postbuild    # Run pagefind indexer after build (required for search)
pnpm preview      # Preview production build locally
pnpm lint         # Biome lint
pnpm format       # Full format (biome format + prettier + import sorting)
pnpm check        # Astro type check
```

Pagefind search only works after a full `build && postbuild` — it has no dev-mode equivalent.

## Architecture

This is an **Astro v5** blog (based on the Astro Citrus template) deployed to Netlify. The site is in Traditional Chinese (zh-TW) and lives at `https://www.billyji.com/`.

### Content Collections

Three collections defined in `src/content.config.ts`:

- **post** — `src/content/post/` — full blog posts. Each post is a folder with `index.md` (or `.mdx`) and any accompanying images. Key frontmatter: `title`, `description`, `publishDate`, `tags`, `draft`, `coverImage`, `ogImage`, `seriesId`, `orderInSeries`.
- **note** — `src/content/note/` — shorter notes. Can be a single `.md` file or a folder.
- **series** — `src/content/series/` — groups posts by `seriesId`. A series file has `id`, `title`, `description`, `featured`.

`draft: true` posts are excluded from production builds, RSS, OG images, and all `getAllPosts()` calls.

### Key Source Paths

- `src/site.config.ts` — site metadata (`author`, `title`, `description`, `lang`) and `menuLinks`.
- `src/styles/global.css` — CSS variables for light/dark themes.
- `src/plugins/` — two custom remark plugins: `remark-reading-time.ts` and `remark-admonitions.ts` (handles `:::note`, `:::tip`, etc. directives).
- `src/pages/og-image/[slug].png.ts` — Satori-based OG image generation per post.
- `src/pages/rss.xml.ts` — RSS feed.

### Markdown Pipeline

Syntax highlighting uses **rehype-pretty-code** (Shiki), with `rose-pine` (dark) and `rose-pine-dawn` (light) themes. Changing the theme requires a dev server restart. Admonitions use `:::type` directive syntax via `remark-directive` + the custom `remarkAdmonitions` plugin.

### Tooling

- **Biome** — linting and formatting for non-Astro files (tabs, indent width 2, line width 100).
- **Prettier** — formatting for `.astro` files and others not covered by Biome.
- **Pagefind** — static search, scoped to `data-pagefind-body` elements in `BlogPost.astro` and `Note.astro`.
- **pnpm** — package manager (see `pnpm-lock.yaml`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yaj55billy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

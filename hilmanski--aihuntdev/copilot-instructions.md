## aihuntdev

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIHunt.dev - A curated directory for AI developer tools, built with Astro (v5) as a static site.

## Development Commands

```bash
npm run dev      # Start dev server at localhost:4321
npm run build    # Build production site to ./dist/
npm run preview  # Preview production build locally
```

## Architecture

**Framework:** Astro 5 with MDX support for content

**Content System:**
- Tools live in `src/content/tools/` as Markdown/MDX files
- Schema defined in `src/content.config.ts` using Zod validation
- Frontmatter structure:
  ```yaml
  title: string          # Tool name
  description: string    # Brief description of the tool
  createdAt: date        # Date added (YYYY-MM-DD)
  logo: ""               # Logo URL (default: "")
  website: ""            # Tool website URL (default: "")
  category:              # Array of category strings
    - "Category 1"
    - "Category 2"
  ```

**Routing:**
- File-based routing in `src/pages/`
- Dynamic tool routes via `src/pages/tools/[...slug].astro`
- RSS feed generated at `src/pages/rss.xml.js`

**Key Patterns:**
- Layout composition: `ToolPage.astro` layout wraps content with header/footer
- Scoped CSS in `.astro` files, global styles in `src/styles/global.css`
- SEO meta tags centralized in `src/components/BaseHead.astro`

**Components for MDX:**
- `Youtube` - Embed YouTube videos in MDX files:
  ```mdx
  import Youtube from '../../components/Youtube.astro';

  <Youtube src="VIDEO_ID" />
  ```

**Integrations:**
- `@astrojs/mdx` - MDX content support
- `@astrojs/sitemap` - Auto sitemap generation
- `@astrojs/rss` - RSS feed generation
- `sharp` - Image optimization

**CSS**
Use Tailwind v4 for styling

## TypeScript

Strict mode enabled. Types extend `astro/tsconfigs/strict`. Generated types in `.astro/types.d.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hilmanski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

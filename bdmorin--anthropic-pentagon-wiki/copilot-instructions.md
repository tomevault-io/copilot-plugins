## anthropic-pentagon-wiki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A wiki-style static site documenting the Anthropic-Pentagon relationship and related AI-military developments. Built with Astro 5, MDX content collections, Tailwind CSS v4, and deployed to Vercel.

## Commands

```bash
npm run dev        # Dev server at localhost:4321
npm run build      # Production build to ./dist/
npm run preview    # Preview production build locally
```

No test framework is configured.

## Architecture

### Content Collections (src/content/)

Six Astro content collections defined in `src/content/config.ts`:

- **people**, **entities**, **events**, **concepts** — MDX content collections with cross-references via Astro's `reference()` system
- **sources** — MDX collection for citations/references
- **timeline** — YAML data collection (not MDX), entries are numbered files like `001-contracts.yaml`

Collections heavily cross-reference each other through frontmatter arrays (e.g., `affiliations`, `keyPeople`, `relatedEvents`, `sources`). These references use Astro's built-in `reference()` type which validates at build time.

### Key Patterns

- **Slug handling**: Astro 5 includes file extensions in `entry.id`. The `slugFromId()` helper in `src/lib/collections.ts` strips `.mdx`/`.yaml`/etc. All page routes use this function.
- **Backlinks**: `src/lib/backlinks.ts` scans all collections at build time to find reverse references — any page that references a target page via frontmatter arrays.
- **Reference resolution**: `resolveRefs()` in `src/lib/collections.ts` converts arrays of `{ collection, id }` references into full collection entries.
- **Dynamic routes**: Each collection has `src/pages/[collection]/[...slug].astro` using `getStaticPaths()`. Each resolves its own references and renders via the WikiPage layout.

### Layouts

- **BaseLayout** — HTML shell, global styles, meta tags
- **WikiPage** — Three-column layout: sidebar navigation, main content area, table of contents. Used by all collection pages.
- **LandingPage** — Homepage layout

### Integrations

- **astro-pagefind** — Full-text search indexed at build time
- **@astrojs/react** — Used for the Timeline interactive component (`src/components/Timeline.tsx`)
- **satori + sharp** — OG image generation at `src/pages/og/[...slug].png.ts`
- **Tailwind CSS v4** — Via Vite plugin (not Astro integration)

### Adding Content

New wiki entries are MDX files in the appropriate `src/content/` subdirectory. Frontmatter must match the Zod schema in `config.ts`. Reference other entries by their filename (without extension) using the collection name, e.g., `sources: [techcrunch-feb-15]`.

Timeline entries are YAML files numbered sequentially (e.g., `011-new-event.yaml`) with an optional `linkedEvent` reference.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bdmorin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bdmorin)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

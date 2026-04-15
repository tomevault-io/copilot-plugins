## nestledjs-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```shell
pnpm dev          # Start dev server (localhost:3001)
pnpm build        # Production build (also validates all routes)
pnpm lint         # ESLint
pnpm lint:fix     # ESLint with auto-fix
pnpm type-check   # TypeScript type checking (tsc --noEmit)
pnpm format       # Prettier format all files
pnpm format:check # Prettier check (CI)
pnpm test         # lint + type-check + build (no unit tests)
pnpm check-links  # Validate internal links across doc pages
```

Pre-commit hook runs `lint-staged` (Prettier) and `check-links` automatically.

## Architecture

This is the **documentation site for Nestled** (nestledjs.com), built with Next.js 15 and Markdoc.

### Content System

Documentation pages are **Markdoc `.md` files** that Next.js treats as pages (via `pageExtensions: ['md', ...]` in `next.config.mjs`). Each doc lives at `src/app/docs/<slug>/page.md`.

- `src/markdoc/nodes.js` — custom Markdoc node renderers (document → `DocsLayout`, fence → `Fence`)
- `src/markdoc/tags.js` — custom Markdoc tags: `{% callout %}`, `{% quick-links %}`, `{% quick-link %}`, `{% figure %}`
- `src/markdoc/search.mjs` — build-time search index generation

### Markdoc Code Block Requirement

**Every fenced code block MUST have a language identifier** (e.g., ` ```shell `, ` ```typescript `, ` ```text `). A bare ` ``` ` without a language crashes `prism-react-renderer` at runtime. Use `text` for plain output or tree structures.

### Navigation & Layout

- `src/lib/navigation.ts` — defines the sidebar link order (single flat "Documentation" group)
- `src/lib/sections.ts` — extracts H2/H3 headings for the right-side table of contents
- `src/components/Layout.tsx` → `DocsLayout` wraps all pages with sidebar, TOC, and prev/next links

### Page Frontmatter

Every `page.md` uses YAML frontmatter with a `title` and a `nextjs.metadata` block:

```yaml
---
title: Page Title
nextjs:
  metadata:
    title: Page Title
    description: A rich description for SEO and AI search snippets.
---
```

### LLM SEO Files

All LLM SEO routes auto-discover pages via the shared `src/lib/docs.ts` utility (globs `**/page.md`). **No manual updates needed when adding or removing doc pages** — just add the `page.md` file and an entry in `src/lib/navigation.ts`.

- `src/lib/docs.ts` — shared page discovery; all routes below use `getDocPages()`
- `src/app/llms.txt/route.ts` — summary with links for LLM crawlers
- `src/app/llms-full.txt/route.ts` — concatenated full docs for LLM crawlers
- `src/app/sitemap.ts` — XML sitemap
- `src/app/robots.ts` — static config

### Path Alias

`@/*` maps to `./src/*` (tsconfig paths).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nestledjs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nestledjs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

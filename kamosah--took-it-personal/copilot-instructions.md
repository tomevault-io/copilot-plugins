## took-it-personal

> Personal portfolio and blog built with Next.js 16 App Router, React 19, Tailwind CSS v4, and Contentlayer2.

@AGENTS.md

# took-it-personal

Personal portfolio and blog built with Next.js 16 App Router, React 19, Tailwind CSS v4, and Contentlayer2.

## Commands

```bash
npm run dev          # start dev server (runs contentlayer2 build first)
npm run build        # production build (runs contentlayer2 build first)
npm run lint         # ESLint (Next.js core web vitals + TypeScript rules)
npm run typecheck    # tsc --noEmit
npm run clean        # remove .next and .contentlayer
```

Run `lint` and `typecheck` before committing. CI enforces both on every push and PR to `main`.

## Project structure

```
src/
├── app/              # Next.js App Router pages
├── components/
│   ├── layout/       # Structural shells (Nav, Footer, ThemeProvider)
│   ├── sections/     # Full-width, page-level compositions (Hero, FeaturedWork)
│   ├── ui/           # Reusable primitives (Button, Badge, Card)
│   └── mdx/          # Blog-specific component overrides
├── content/posts/    # MDX blog posts
├── lib/              # Utilities (contentlayer helpers, cn())
└── types/            # Shared TypeScript interfaces
```

## Content (Contentlayer2)

Blog posts live in `src/content/posts/*.mdx`. Required frontmatter:

```mdx
---
title: string
description: string
date: YYYY-MM-DD
tags: [string]       # optional
published: boolean   # defaults to true
---
```

Computed fields (`slug`, `readingTime`, `url`) are auto-generated — do not add them to frontmatter.

Import posts via `src/lib/contentlayer.ts`, not directly from `contentlayer/generated`.

## Styling

Tailwind CSS v4 with design tokens defined in `src/app/globals.css` via `@theme`. Always use token-based classes (`bg-bg`, `text-text-primary`, `text-accent`) rather than raw Tailwind color utilities. Light/dark mode is handled via the `.dark` class selector (`next-themes`).

Use the `cn()` utility from `src/lib/utils.ts` for conditional class merging.

## Key conventions

- Path alias `@/*` resolves to `src/*`
- Components: PascalCase filenames
- Utilities/hooks: camelCase filenames
- `'use client'` only where interactivity or browser APIs are needed — keep server components as the default
- Animations use Framer Motion; `AnimatedSection` handles staggered reveal patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamosah)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kamosah)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

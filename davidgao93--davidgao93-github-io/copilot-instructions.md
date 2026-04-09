## davidgao93-github-io

> Single-page portfolio/resume website built with Next.js 14 (static export), React 18, TypeScript, and Tailwind CSS. Hosted on GitHub Pages.

# CLAUDE.md

## Project Overview

Single-page portfolio/resume website built with Next.js 14 (static export), React 18, TypeScript, and Tailwind CSS. Hosted on GitHub Pages.

## Commands

```bash
yarn dev        # Development server
yarn build      # Production build (tsc + next build)
yarn lint       # Prettier format + ESLint fix
yarn clean      # Remove .next and build-tsc dirs
yarn sitemap    # Generate sitemap
```

## Architecture

**Content lives in `src/data/`** — edit `data.tsx` to update site content, `dataDef.ts` for TypeScript interfaces. Components should not contain content directly.

**Component structure:**
- `src/components/` — all React components, organized by section
- `src/pages/index.tsx` — single page, composes all sections
- `src/hooks/` — custom hooks (window, nav observer, outside click, interval)
- `src/config.ts` — browser/device detection utilities

## Code Conventions

- All components must be wrapped in `React.memo()` — ESLint enforces this
- Functional components with TypeScript; no prop-types
- Tailwind for utility classes, SCSS for global/complex styles
- Import order enforced by `simple-import-sort` ESLint plugin
- Strict TypeScript: `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`
- Prettier: 2-space indent, 120 char width, single quotes, no bracket spacing
- `Header` component uses `dynamic()` with `ssr: false` (client-only)

## Key Files

| File | Purpose |
|------|---------|
| `src/data/data.tsx` | All site content (text, links, images) |
| `src/data/dataDef.ts` | TypeScript interfaces for data shapes |
| `src/globalStyles.scss` | Global styles |
| `tailwind.config.js` | Custom colors (`yellow: #efc603`) and animations |
| `next.config.js` | Static export config, Unsplash image remote pattern |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidgao93)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidgao93)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

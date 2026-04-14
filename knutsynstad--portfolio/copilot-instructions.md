## portfolio

> Project overview and coding conventions


# Portfolio — Knut Synstad

Next.js 16 App Router portfolio site. Static pages, no dynamic routes.

## Stack

- Next.js 16 (Turbopack), React 19, TypeScript (strict)
- Tailwind CSS 4 (CSS-first config in `globals.css`)
- ESLint 9 (flat config), Prettier 3

## Structure

- `app/` — pages, layouts, components, styles
- `app/components/` — shared UI components
- `app/projects/<slug>/page.tsx` — one file per project
- `public/` — static assets (images, videos, files)
- Path alias: `@/*` maps to `./app/*`

## Conventions

- Be concise. Prefer clarity over cleverness.
- Server Components by default; add `"use client"` only when needed.
- Run `npm run check` before considering work done.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/knutsynstad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/knutsynstad)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

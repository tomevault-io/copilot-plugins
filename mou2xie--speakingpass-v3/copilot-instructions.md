## speakingpass-v3

> - `npm run dev` — dev server with Turbopack

# AGENTS.md

## Quick commands
- `npm run dev` — dev server with Turbopack
- `npm run build` — production build (runs `next-sitemap` as `postbuild`)
- `npm run lint` — ESLint
- There is no `test` or `typecheck` script

## Tech stack
Next.js 15 App Router, React 19, TypeScript strict, Tailwind CSS v4, DaisyUI v5, Supabase, Remix Icon, Zustand.

## Architecture

### Component/server boundary
- All components in `src/components/` and pages in `src/app/` are **server components by default**.
- Only `NavBar.tsx` and `MobileNavMenu.tsx` are marked `'use client'` (for scroll listener and toggle state).
- **Do not add `'use client'` to new components unless they need browser APIs or interactivity.**

### Data fetching
- Supabase client in `src/libs/supabase.ts` is wrapped with `import 'server-only'` — it can only be used in server components or server actions.
- All data queries live in `src/libs/actions.ts` as server functions (marked `'use server'`).
- Supabase tables: `v3_part1_category`, `v3_part1_topic`, `v3_part2_topic`.

### URL encoding
- Custom URL-friendly encoding in `src/libs/functions.ts`: spaces ↔ hyphens. **Do not use `encodeURIComponent` for route segments.**
- `encodeURLSegment(str)` replaces `/\s+/g` with `-`. `decodeURLSegment(str)` reverses it.

### Content model
- Topics are tagged with `type`: `'CURRENT'`, `'MUST'`, or `null` (past).
- The current season string (`"Jan - Apr"`, `"May - Aug"`, `"Sep - Dec"`) is computed at build time from `dateUtils.ts` and used via `CURRENT_MONTH` in `currentMonth.ts`. **Not an env var — it's date-based.**

### Styling
- Tailwind v4 + DaisyUI v5 configured in `globals.css` via `@import "tailwindcss"` and `@plugin "daisyui"`.
- Custom colors defined in the `@theme` block: `blue-primary`, `text-strong`, `text-main`, `text-light`, `grey`.
- Custom fonts via `next/font/google` in root layout: PT Serif (`--font-ptSerif`), Bree Serif (`--font-breeSerif`), Oswald (`--font-oswald`).
- Use `font-(family-name:--font-breeSerif)` syntax for custom font families (Tailwind v4 arbitrary property).

### Sitemap
- `next-sitemap.config.js` is a **CommonJS** file (uses `require`), unlike the rest of the codebase (ESM).
- Generated sitemap excludes `/part1/*` and `/part2/question/*`, then dynamically adds them from Supabase at build time.

### Zustand
- Listed as a dependency but currently unused in the codebase. Keep if adding client-side state.

## Environment
- `.env` is committed to the repo (real credentials). The `.env*` gitignore pattern does not match `.env` without a suffix.
- Required keys: `SUPABASE_URL`, `SUPABASE_KEY`.

---
> Source: [Mou2xie/SpeakingPass_v3](https://github.com/Mou2xie/SpeakingPass_v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->

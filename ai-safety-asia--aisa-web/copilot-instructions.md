## aisa-web

> AI Safety Asia public website (https://www.aisafety.asia/) — the org's front-facing site

# AGENTS.md — aisa-web

AI Safety Asia public website (https://www.aisafety.asia/) — the org's front-facing site
presenting its mission, programs, team, insights, and articles. (Root layout metadata title
is "AI Safety Asia".)

## Stack
- Next.js ^16.1.4 (App Router) + React 19.1.0
- TypeScript ^5 (strict)
- Tailwind CSS v4 via `@tailwindcss/postcss` (no `tailwind.config` file)
- react-icons, `@vercel/analytics`

## Commands
- `npm run dev` — dev server (`next dev`)
- `npm run build` — production build (`next build`)
- `npm run start` — serve production build (`next start`)
- `npm run lint` — ESLint via `next lint` (extends `next/core-web-vitals`, `next/typescript`)
- No test script is defined.

## Layout (all under `src/`)
- `src/app/` — App Router routes; each page is a `page.tsx` (e.g. `who-we-are/`,
  `what-we-do/`, `insights/`, `articles/`, `news/`). `layout.tsx` is the root shell.
- `src/app/**/sections/` — page-specific section components (Hero, Mission, Team, …).
- `src/components/` — shared UI components (Header, Footer, ProfileCard, cards, wrappers).
- `src/data/` — static content as typed TS modules (team, advisors, articles, insights,
  navItems, topics, videos, …); most page copy/data lives here, not in a CMS.

## Notes
- Import alias `@/*` → `./src/*` (`tsconfig.json`).
- Global styles: `src/app/globals.css`, imported in `layout.tsx`; fonts are Geist / Geist
  Mono via `next/font/google`.
- `next.config.ts` is effectively empty (framework defaults).

Org context: see ../AGENTS.md (AISA workspace root).

---
> Source: [AI-Safety-Asia/aisa-web](https://github.com/AI-Safety-Asia/aisa-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

## codejeet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CodeJeet is a fully static Next.js 16 app for browsing company-wise LeetCode DSA questions and system design chapters. It uses shadcn/ui components and reads question data from CSV files at build time. Deployed to Cloudflare Pages — no server required.

## Commands

- **Dev:** `pnpm dev` (uses Turbopack)
- **Build:** `pnpm build` (runs prebuild script then `next build`, outputs to `out/`)
- **Prebuild:** `pnpm run prebuild` (generates `public/data/questions.json` from CSVs)
- **Lint:** `pnpm lint` / `pnpm lint:fix`
- **Format:** `pnpm format` (Prettier) / `pnpm format:check`
- **Pre-commit hook:** Husky runs lint-staged (Prettier auto-format) on commit

## Architecture

### Data Layer — CSV-based, prebuild to static JSON

All question data lives in `data/*.csv` (one CSV per company, ~200+ files). At build time, `scripts/build-data.ts` uses `lib/data.ts` to parse all CSVs and writes `public/data/questions.json` containing `{ questions, companies, topics }`. This JSON file is served statically and fetched by the dashboard client.

### No API Routes

There are no API routes. The dashboard fetches `/data/questions.json` directly. The prebuild step (`tsx scripts/build-data.ts`) generates this file before `next build`.

### Client-side Caching

The dashboard client (`app/dashboard/page.client.tsx`) caches the JSON response in `localStorage` keyed by `CACHE_VERSION` from `lib/cache-version.ts`. When updating the data or JSON shape, bump `CACHE_VERSION` to bust client caches.

### Dashboard (Server + Client split)

`app/dashboard/page.tsx` is a statically rendered server component (`force-static`) that renders `page.client.tsx`. The client component fetches `/data/questions.json` and renders `LeetCodeDashboard`.

### System Design Pages

Markdown-based content in `public/system-design/` with numbered folders (e.g., `01. Scaling/README.md`). Each markdown file uses gray-matter frontmatter with `slug`, optional `video` (YouTube), and `podcast` (Spotify) fields. Pages are statically generated (`force-static`, `dynamicParams: false`).

### No Auth

The app is fully public — no authentication layer. All pages are accessible without sign-in.

### Static Export

Next.js `output: "export"` produces a fully static `out/` directory. No Node.js server needed. Deployed to Cloudflare Pages.

### UI Stack

- shadcn/ui (new-york style) with Radix primitives — components in `components/ui/`
- Tailwind CSS v3 with oklch color tokens defined in `app/globals.css`
- Magic UI animation components in `components/magic-ui/`
- Fonts: Plus Jakarta Sans (sans) + JetBrains Mono (mono)
- Dark theme default via `next-themes`

### Path Aliases

`@/*` maps to project root (e.g., `@/components`, `@/lib`, `@/utils`, `@/hooks`).

## Key Conventions

- Package manager is **pnpm** (specified in `packageManager` field)
- Next.js output mode is `export` (fully static)
- Prettier config: double quotes, semicolons, 100 char width, es5 trailing commas
- `utils/utils.ts` has general helpers (e.g., `capitalizeWords`); `lib/utils.ts` has the shadcn `cn()` merge utility
- `public/data/` is gitignored — generated at build time from CSVs

---
> Source: [ayush-that/codejeet](https://github.com/ayush-that/codejeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

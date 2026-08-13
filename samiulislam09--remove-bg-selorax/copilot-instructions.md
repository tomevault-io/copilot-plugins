## remove-bg-selorax

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A background removal tool built as a SeloraX platform app. Next.js 16 frontend (React 19, TypeScript, Tailwind CSS v4) using the App Router.

## Commands

```bash
bun dev       # Dev server at http://localhost:3000
bun run build # Production build
bun run start # Start production server
bun run lint  # ESLint (next/core-web-vitals + next/typescript)
```

Package manager is **Bun** (bun.lock present). No test runner is configured.

## Architecture

- **App Router** — `app/` directory with `layout.tsx` (root layout, Geist fonts) and `page.tsx`
- **Styling** — Tailwind CSS v4 via `@tailwindcss/postcss` plugin; theme variables defined in `app/globals.css` using `@theme inline`
- **Path alias** — `@/*` maps to project root (tsconfig paths)

## SeloraX Platform Integration

This app authenticates with the SeloraX platform using client credentials in `.env`:
- `SELORAX_CLIENT_ID` / `SELORAX_CLIENT_SECRET` — app identity for platform API calls
- `SELORAX_APP_ID` — registered app ID on the platform
- `SESSION_SIGNING_KEY` — optional, for local JWT verification (falls back to platform verification if unset)

---
> Source: [samiulislam09/remove-bg-selorax](https://github.com/samiulislam09/remove-bg-selorax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

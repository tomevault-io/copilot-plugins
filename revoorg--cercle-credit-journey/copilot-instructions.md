## cercle-credit-journey

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cercle is a Progressive Web App showcasing future credit journeys. It's a frontend-only demo — no real backend logic. The app follows scripted scenarios to simulate realistic credit experiences.

## Commands

- `npm run dev` — Start dev server (Turbopack)
- `npm run build` — Production build
- `npm run start` — Serve production build
- `npm run lint` — ESLint (flat config, Next.js core-web-vitals + TypeScript rules)

No test framework is configured yet.

## Architecture

- **Next.js 16** with App Router (not Pages Router) and React 19
- **TypeScript** in strict mode, path alias `@/*` → `./src/*`
- **Tailwind CSS v4** with OKLCH color tokens defined as CSS variables in `src/app/globals.css`
- **shadcn/ui** (New York style) with Radix UI primitives and Lucide icons — add components via `npx shadcn@latest add <component>`
- **Geist** font family (sans + mono) loaded via `next/font/google`

## PWA Setup (manual, no third-party plugin)

- `public/manifest.json` — Web app manifest (standalone display, portrait)
- `public/sw.js` — Service worker with network-first strategy + cache fallback
- `src/components/service-worker-register.tsx` — Client component that registers the SW on mount
- `src/app/offline/page.tsx` — Offline fallback page
- Icons in `public/icons/` (192, 512, maskable variants) + `public/apple-touch-icon.png`
- PWA meta tags are set via Next.js Metadata API in `src/app/layout.tsx`

When modifying the service worker cache strategy or adding new offline-capable routes, update the `CACHE_NAME` version in `sw.js` to bust stale caches.

## Conventions

- Dark mode uses `.dark` class toggle with CSS custom properties
- All colors use OKLCH color space
- Use the `cn()` helper from `@/lib/utils` for conditional Tailwind classes
- React Server Components by default; add `"use client"` only when needed
- Viewport is locked to portrait, no user scaling (mobile-app feel)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/revoorg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

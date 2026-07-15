## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for a DevOps engineer, built with Nuxt 3 (Vue 3) and TypeScript. The site is in French. Live at https://champagnedevops.fr.

## Commands

```bash
npm install           # Install dependencies
npm run dev           # Start dev server (with HMR)
npm run build         # Production build (SSR + static generation)
npm run generate      # Static site generation only
npm run preview       # Preview production build locally
```

No test framework or linter is configured.

## Architecture

**Framework**: Nuxt 3 with SSR enabled. Pages `/`, `/about`, `/projects`, `/contact` are pre-rendered. Dynamic project pages use `pages/project/[slug].vue`.

**Key libraries**: Three.js (3D particle background), GSAP + ScrollTrigger (animations), Tailwind CSS (styling), Pinia (state), marked (markdown rendering), @vueuse/core (composables).

**Data flow**: Project data lives in `data/projects.js` as a static array. Each project has a `slug` and optional `markdownFile` referencing files in `public/projects/*.md` that are fetched and rendered with `marked`.

**Plugins** (all client-side only, `.client.ts`):
- `gsap.client.ts` — registers ScrollTrigger plugin
- `navigation-tracking.client.ts` — GA4 page view tracking
- `scroll-tracking.client.ts` — scroll depth and section visibility tracking
- `pwa-manual.client.ts` — service worker lifecycle management

**Composables**: `useEnhancedTracking` and `useNavigationTracking` wrap Google Analytics 4 event tracking.

**Build optimization**: Manual Vite chunks split `vue`/`vue-router`, `three`, `gsap`, and `@vueuse/core` into separate bundles. Three.js and GSAP are transpiled via `build.transpile`.

## Deployment

Pushes to `main` trigger GitHub Actions CI/CD (`.github/workflows/deploy.yml`):
1. Build with Node 20 and `npm ci --frozen-lockfile`
2. rsync `.output/public/` to VPS at `/srv/www/portfolio/releases/<timestamp>/`
3. Atomic symlink swap to `/srv/www/portfolio/current`
4. Keeps 3 most recent releases for rollback

Web server is Caddy with automatic TLS, gzip/zstd compression, and long-lived asset caching.

## Conventions

- All Vue components use `<script setup>` with Composition API
- Components are auto-imported (no explicit import statements needed)
- Tailwind CSS for styling with a custom `primary` color (indigo palette: 300/#a5b4fc, 400/#818cf8, 500/#6366f1)
- GA4 page views are manually managed (`send_page_view: false` in config) via plugins and composables
- Use `data-section="section-name"` attributes on elements for automatic GA4 section tracking
- Content language is French throughout

---
> Source: [CaptainIgl00/portfolio](https://github.com/CaptainIgl00/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->

## erealogistics-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Brand Identity

All design and copy decisions must follow **[BRAND.md](BRAND.md)**. Read it before any visual, layout, component, or content work.

## Project Overview

EREA Logistics Services — a bilingual (ES/EN) single-page marketing site for a senior supply chain consulting practice. Static site deployed to erealogistics.com.

## Commands

- `npm run dev` — Start dev server (Vite)
- `npm run build` — Production build (static output to `build/`)
- `npm run preview` — Preview production build locally
- `npm run check` — Type-check with svelte-check

No test runner is configured.

## Tech Stack

- **SvelteKit 2** (Svelte 5 with runes) + `adapter-static` for fully prerendered output
- **Tailwind CSS v4** via `@tailwindcss/vite` plugin — config lives in `src/app.css` using `@theme` and `@utility` directives (no `tailwind.config`)
- **GSAP + ScrollTrigger** for scroll-driven animations
- **TypeScript** with strict mode

## Architecture

### Single-page structure
The site is a single route (`src/routes/+page.svelte`) with section-based navigation. All pages are prerendered (`export const prerender = true`) with trailing slashes.

### i18n system
- Content lives in `src/lib/content/en.json` and `src/lib/content/es.json`
- `src/lib/content/types.ts` derives the `Content` type from the English JSON
- `src/lib/stores/lang.svelte.ts` exposes `lang` (a Svelte 5 rune-based store class) and `getContent()`
- Language is detected from `?lang=` URL param → localStorage → defaults to `es`
- Language toggle uses View Transitions API when available, with scroll-anchor correction to prevent drift

### Animation system
- `src/lib/animations/gsap.ts` — Core animation functions (`revealOnScroll`, `staggerReveal`, `lineReveal`, `ambientParallax`, `countUpOnScroll`) with shared constants for easing, duration, offsets, stagger, and trigger positions
- `src/lib/animations/actions.ts` — Svelte actions wrapping GSAP functions for declarative `use:reveal`, `use:lineReveal`, `use:stagger`, `use:parallax`, `use:countUp` directives
- All animations respect `prefers-reduced-motion` by snapping to final state
- ScrollTriggers are cleaned up in `+layout.svelte` on unmount

### Design system (in `src/app.css`)
- Custom `@theme` block defines the full color system, typography, spacing, and shadows
- Custom `@utility` directives define reusable patterns: `px-container`, `py-section`, `text-hero`, `text-h2`, `eyebrow`, `section-rule`, `surface-card`, `button-primary`, `metric-chip`, etc.
- Supply chain document aesthetic classes: `.manifest-ref-tag`, `.manifest-line-item`, `.kpi-tile`, `.delivery-proof-card`, `.audit-checklist`, `.workflow-steps`
- **Always use CSS custom properties** (`var(--color-*)`) — never inline hex colors

### Build optimizations
Manual chunks in `vite.config.ts` split `gsap` into a separate bundle.

### Fonts
Custom fonts (Equity serif, Concourse sans-serif) served from `static/fonts/` as woff2.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ereaxp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ereaxp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

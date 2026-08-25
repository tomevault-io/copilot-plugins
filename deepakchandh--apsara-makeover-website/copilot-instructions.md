## apsara-makeover-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-page marketing website for **Apsara Makeover**, a beauty parlour in Madurai, India. Built with v0.app and deployed on Vercel. The site is SEO-focused with structured data markup targeting local search queries.

## Commands

- `npm run dev` — Start development server (Next.js)
- `npm run build` — Production build (TypeScript errors are ignored via `next.config.mjs`)
- `npm run lint` — ESLint
- `npm start` — Start production server

No test framework is configured.

## Architecture

**Framework:** Next.js 16 (App Router) with React 19, TypeScript, Tailwind CSS v4

**Single-page structure:** `app/page.tsx` composes all sections in order:
`SchemaMarkup → Header → Hero → About → Services → Reviews → FAQ → Contact → Footer`

Each section is a standalone component in `components/`. Navigation uses anchor links (`#home`, `#about`, `#services`, `#reviews`, `#contact`).

**UI components:** shadcn/ui (new-york style) in `components/ui/`. Configured via `components.json`. Uses Radix UI primitives, `class-variance-authority`, and `lucide-react` icons.

**Styling:**
- Tailwind CSS v4 with `@tailwindcss/postcss` (not legacy `tailwindcss` config file — config is in `app/globals.css`)
- CSS custom properties in oklch color space defined in `app/globals.css` (`:root` and `.dark` variants)
- Two font families: Inter (`--font-inter`, `font-sans`) for body, Playfair Display (`--font-playfair`, `font-serif`) for headings
- Utility: `cn()` from `lib/utils.ts` (clsx + tailwind-merge)

**SEO:** `components/schema-markup.tsx` injects JSON-LD for `BeautySalon` (LocalBusiness) and `FAQPage` schemas. Metadata (OpenGraph, canonical URL, keywords) is in `app/layout.tsx`.

**Key config notes:**
- `typescript.ignoreBuildErrors: true` in `next.config.mjs`
- `images.unoptimized: true` — images served without Next.js optimization
- Path alias: `@/*` maps to project root

**Client components:** Only `Header` (mobile menu state) and `FAQ` (accordion interaction) use `"use client"`. All other section components are server components.

## Import Conventions

- Use `@/` path alias for all imports (e.g., `@/components/ui/button`, `@/lib/utils`)
- shadcn/ui components: `@/components/ui/<component>`
- Custom hooks: `@/hooks/<hook>`

---
> Source: [deepakchandh/apsara-makeover-website](https://github.com/deepakchandh/apsara-makeover-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

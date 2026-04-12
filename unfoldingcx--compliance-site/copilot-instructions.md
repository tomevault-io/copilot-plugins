## compliance-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-tenant compliance center SPA that displays Q&A-style compliance information (policies, procedures, security guidelines). Supports multiple languages and domain-based branding. Deployed on Netlify.

## Commands

- `npm run dev` — Start Vite dev server
- `npm run build` — TypeScript check + Vite production build (`tsc -b && vite build`)
- `npm run lint` — ESLint (flat config, TypeScript + React hooks + React Refresh)
- `npm run preview` — Preview production build locally

No test framework is configured.

## Architecture

### Tech Stack
React 18 + TypeScript + Vite, Tailwind CSS v3 (with shadcn/ui components via Radix primitives), Zustand for state, TanStack React Query (configured in `main.tsx`), Framer Motion for animations.

### Path Alias
`@/*` maps to `./src/*` (configured in both `tsconfig.json` and `vite.config.ts`).

### Key Data Flow

**Compliance data** is loaded per-language from static JSON files at `/public/translations/{locale}.json` (e.g. `en_US.json`, `pt_BR.json`). Each file is an array of `{ category, question, answer }` objects. The `useComplianceData` hook (`src/hooks/useComplianceData.ts`) fetches this based on the current language from the Zustand store.

**Branding** is multi-tenant via hostname. `public/branding/config.json` maps domain names to branding config (companyName, themeColor, etc.). Each domain has its own directory under `public/branding/{domain}/` with logo and favicon assets. The `useBranding` hook (`src/hooks/useBranding.ts`) loads the config, matches `window.location.hostname`, and dynamically updates document title, meta tags, favicon, and CSS custom properties (`--theme-color`, `--theme-color-rgb`). Localhost always uses the `default` branding.

**Global state** lives in a single Zustand store (`src/store/complianceStore.ts`): `activeCategory`, `activeCategoryIndex`, `searchQuery`, and `language` (type `LanguageCode`).

### Translation System

Two separate translation mechanisms exist:
1. **UI translations** — Hardcoded in `src/translations/index.ts` and also duplicated inline in several components (`App.tsx`, `Header.tsx`, `Footer.tsx`, `ComplianceList.tsx`, `CategoryNavigation.tsx`). Accessed via `useTranslation` hook or direct object lookup.
2. **Compliance content** — Loaded from `/public/translations/{locale}.json` via fetch.

Supported locales: `en_US`, `pt_BR`, `fr_FR`, `de_DE`, `es_ES`.

### Component Structure

- `src/components/ui/` — shadcn/ui primitives (accordion, button, card, dialog, etc.)
- `src/components/` — App-specific components (Header, Footer, CategoryNavigation, ComplianceList, ComplianceItem, WelcomeScreen, etc.)
- Category navigation uses a fixed icon array (`categoryIcons` in `CategoryNavigation.tsx`) mapped by index to categories from the data

### Styling

Tailwind with CSS custom properties for theming (light/dark via `next-themes` with class strategy). HSL color tokens defined in `src/index.css`. The branding `themeColor` is injected at runtime as `--theme-color` on the document root.

### Adding a New Tenant

1. Add domain entry to `public/branding/config.json`
2. Create `public/branding/{domain}/` directory with `logo.png` and `favicon/` assets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unfoldingcx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/unfoldingcx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

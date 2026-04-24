## sillylilgoose

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server at http://localhost:3000
npm run build    # Production build
npm run lint     # Run Next.js ESLint
npm run start    # Start production server
```

Node version: 22.x (see `.nvmrc`)

No test suite is configured.

## Architecture

**Stack:** Next.js 15 (App Router), React 19, Firebase 9, SCSS Modules, Sass

**Path aliases** (jsconfig.json):
- `@/*` → `src/*`
- `@lib/*` → `lib/*`
- `@utilities/*` → `src/styles/utility/*`

### App Structure

`src/app/` uses the Next.js App Router. Most components are client components (`'use client'`). Routes:
- `/` — Notes home, fetches from Firestore `notes` collection
- `/Frog` — Interactive frog gallery
- `/Info` — Plans/info with reveal animation, fetches from `info/plans`
- `/Spoopy` — Halloween section
- `/Spoopy/Spider` — Nested route

`src/app/layout.jsx` is the root layout. It wraps the app in `UserContext`, conditionally renders `<Login>` if unauthenticated, otherwise renders `<Header>` + children.

### Firebase & Data

`lib/firebase.ts` — Firebase app initialization (project: `sillygoose-cfc85`), exports `auth`, `firestore`, `storage`.

`lib/functions.ts` — All Firestore queries. Fetches notes, titles, and info content.

`lib/hooks.ts` — Two custom hooks:
- `useUserData()` — wraps `useAuthState` to return `{ user, username }`
- `useCachedData(key, getData)` — localStorage-backed data fetching; initializes from cache, fetches fresh data on cache miss

`lib/context.ts` — React context shape for `{ user, username }`, consumed in layout for auth gating.

Authentication uses a single hardcoded email (`adeline@email.com`) with Firebase email/password auth.

### Theme System

Two themes: `spring` (default) and `halloween`.

`lib/theme.ts` — Theme IDs, metadata (symbols, images, behavior flags like whether stacked notes expand). Theme is also stored in Firestore at `config/current-theme`.

`lib/ThemeContext.tsx` — React context + `useTheme()` hook providing `{ theme, metadata }`.

`src/components/ThemeProvider.jsx` — On mount, reads theme from Firestore and sets `data-theme` attribute on `<html>`. This triggers CSS variable switching.

Theme CSS files live in `src/styles/themes/` (`halloween.scss`, `spring.scss`) and define CSS custom properties scoped to `[data-theme="..."]`.

### Styling

SCSS Modules throughout. Style files organized under `src/styles/`:
- `utility/` — variables (colors, fonts, breakpoints, spacing) and mixins; imported via `@utilities/*` alias
- `foundational/` — global styles, header, loading
- `components/` — component-scoped modules
- `pages/` — page-scoped modules
- `themes/` — theme CSS variable definitions

Global styles are imported in `src/app/layout.jsx` via `globals.scss`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/The-INTJ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

## akhil-vipin-nair-signal-circle13

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

C13 Hub — a Next.js team link aggregation platform (like Linktree for teams). Each team member gets a personalized profile page with an iOS 26-inspired liquid glass UI. An admin interface allows editing profiles, which persists changes by committing to GitHub and triggering Vercel rebuilds.

## Commands

- `npm run dev` — Start dev server (http://localhost:3000)
- `npm run build` — Production build
- `npm run start` — Run production server
- `npm run lint` — ESLint (next/core-web-vitals + TypeScript)

No test framework is configured.

## Architecture

**Stack:** Next.js 16 (App Router), React 19, TypeScript 5, Tailwind CSS 4, Framer Motion, vanilla-tilt.js

**Data flow — GitHub as CMS:**
1. Team member data lives in individual TypeScript files under `src/data/` (e.g., `bharath.ts`)
2. `src/data/team.ts` is the central registry that imports and exports all members
3. `src/lib/data.ts` provides runtime access (`getMember`, `getAllMembers`) and GitHub API helpers (`saveMemberToGitHub`)
4. Admin saves → POST `/api/members` (PIN-authenticated) → commits to GitHub → Vercel auto-rebuilds

**Routing:**
- `/[slug]` — Public profile page (server component, `force-dynamic`)
- `/admin` — PIN-protected admin home (lists members, add new)
- `/admin/[slug]` — Member editor with drag-and-drop link reordering (@dnd-kit)
- `/api/members` — GET/POST API for member data
- `/` redirects to `/bharath`

**Theming system (`src/lib/themes.ts`):**
- 7 themes (noir, midnight, ember, forest, royal, ivory, monochrome) defined as CSS custom property sets
- Applied via `ThemeProvider` component using inline style variables
- Each member can set a `theme` field; defaults to "noir"

**Liquid glass UI (`src/app/globals.css`):**
- 3-layer technique: SVG displacement filter (`LiquidGlassFilter` component) + inset box-shadows + vanilla-tilt 3D glare
- `GlassCard` component wraps all interactive cards with tilt, haptic tap sound, and glass styling
- Materialize keyframe animation sequences profile elements on load with staggered delays

**Key patterns:**
- Server components by default; client components only for interactivity (tilt, animations, admin forms)
- Path alias: `@/*` maps to `src/*`
- Icons: custom brand SVGs in `src/lib/icons.tsx` with Lucide React fallback
- Fonts: local Satoshi WOFF2 files loaded in `src/lib/fonts.ts` via `--font-body` CSS variable
- Haptic tap sound: white noise burst via Web Audio API (`src/lib/sound.ts`)

## Environment Variables

```
GITHUB_TOKEN    # Required for admin saves to GitHub
GITHUB_REPO     # Default: nbharath1306/Cygnal-Circle13
GITHUB_BRANCH   # Default: main
ADMIN_PIN       # Required for admin API authentication
```

## Adding a New Team Member

1. Create `src/data/<slug>.ts` exporting a `TeamMember` object (see `src/data/types.ts` for shape)
2. Import and add to the `members` array in `src/data/team.ts`
3. Or use the admin UI at `/admin` which automates both steps via GitHub commits

---
> Source: [heynbharath/Akhil-Vipin-Nair-Signal-Circle13](https://github.com/heynbharath/Akhil-Vipin-Nair-Signal-Circle13) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->

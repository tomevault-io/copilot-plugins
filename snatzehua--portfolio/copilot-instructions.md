## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands run from the `my-portfolio/` subdirectory:

```bash
npm run dev      # Dev server with Turbopack (hot reload)
npm run build    # Production build
npm run start    # Start production server
npm run lint     # ESLint check
```

There are no tests in this project.

## Architecture

This is a Next.js 15 App Router portfolio site using React 19, TypeScript, Tailwind CSS, and Framer Motion.

**Root layout** (`src/app/layout.tsx`) wraps all pages with NavBar, footer, and Vercel Analytics. All pages use `"use client"` and are heavily animated with Framer Motion scroll/transition effects.

**Pages** (`src/app/`):
- `/` — Home: hero, polaroid navigation cards, intro block, scrolling content block, contact block
- `/about` — Three-section personal story with alternating text/image layout
- `/experience` — Project portfolio and professional timeline
- `/explorations` — Hobbies and creative work
- `/contact` — Contact and social links
- `/backend/api/resume-download` — Server-side API route that streams a resume PDF from Google Drive with a dynamic timestamped filename

**Components** (`src/components/`) are organized one-per-folder. The `MobileWarning` component suggests this is desktop-first; mobile users see a warning overlay.

**Animations**: Framer Motion is used pervasively for scroll-triggered reveals, page transitions, and interactive elements. The `--vh` CSS variable is set dynamically for correct mobile viewport height.

**Path alias**: `@/` maps to `src/` — use this for all imports.

**Styling**: Tailwind with custom CSS variables (`--background`, `--foreground`) and a `bounceLine` keyframe animation. Some class names (overlay, menu-button, burger-menu, etc.) are safelisted in `tailwind.config.ts` to prevent purging.

---
> Source: [snatzehua/portfolio](https://github.com/snatzehua/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->

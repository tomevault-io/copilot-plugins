## portfolio

> Personal portfolio site for Angello Lazar, built with Next.js 14 (App Router).

# Portfolio V4

Personal portfolio site for Angello Lazar, built with Next.js 14 (App Router).

## Tech Stack

- **Framework:** Next.js 14.0.1 (App Router)
- **UI:** React 18.2.0
- **Styling:** Global CSS (`public/css/`) + `globals.css`
- **Linting/Formatting:** Biome (replaces ESLint + Prettier)
- **Git Hooks:** Lefthook (pre-commit: lint/format, pre-push: check + build)

## Project Structure

```
app/          → Next.js App Router pages and layout
hooks/        → Custom React hooks (useClickOutside, useWindowSize)
stores/       → Global state management
ui/
  base/       → Layout primitives (header, footer, preloader)
  components/ → Reusable UI components (banner, popups, thumbnails)
  views/      → Page sections (hero, about, portfolio, resume, etc.)
utils/        → Constants, helpers, and portfolio work data
public/       → Static assets (CSS, fonts, images, files)
```

## Commands

```bash
npm run dev        # Start development server
npm run build      # Production build
npm run start      # Start production server
npm run lint       # Run Biome linter
npm run lint:fix   # Run Biome linter with auto-fix
npm run format     # Format code with Biome
npm run check      # Run all Biome checks (lint + format)
npm run check:fix  # Run all Biome checks with auto-fix
```

## Code Conventions

- Single quotes, semicolons, no trailing commas
- 2-space indentation, 80-char line width
- Components use named functions exported as default
- Path aliases: `@/*` → project root, `@css/*` → `public/css/`
- All components are JavaScript (no TypeScript)
- `'use client'` directive on components using browser APIs or React hooks

## Git Hooks (Lefthook)

- **pre-commit:** Runs `biome check --write` on staged files (auto-fixes and re-stages)
- **pre-push:** Runs full `biome check` and `next build` to catch errors before pushing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alo-code) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

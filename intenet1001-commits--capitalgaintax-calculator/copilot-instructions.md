## capitalgaintax-calculator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Capital Gains Tax Calculator - A Next.js 16 web application using React 19, TypeScript, Tailwind CSS 4, and shadcn/ui component library.

## Development Commands

```bash
npm run dev      # Start development server (localhost:3000)
npm run build    # Build for production
npm run start    # Run production build
npm run lint     # Run ESLint
```

## Tech Stack

- **Framework:** Next.js 16.1.1 with App Router and React Server Components
- **UI:** React 19.2.3, Tailwind CSS 4, shadcn/ui (new-york style)
- **Language:** TypeScript 5 (strict mode)
- **Icons:** Lucide React
- **Utilities:** clsx, tailwind-merge, class-variance-authority

## Architecture

### Directory Structure

- `app/` - Next.js App Router pages and layouts
- `lib/` - Utility functions (includes `cn()` for class merging)
- `components/` - React components (to be created)
- `components/ui/` - shadcn/ui components (to be created)
- `hooks/` - Custom React hooks (to be created)
- `public/` - Static assets

### Import Aliases

All configured in tsconfig.json:
- `@/*` - Project root
- `@/components` - Components directory
- `@/ui` - UI components (shadcn)
- `@/lib` - Utility library
- `@/utils` - lib/utils.ts
- `@/hooks` - Custom hooks

## Styling

- Tailwind CSS 4 with CSS variables for theming
- Dark mode via `.dark` class selector
- OKLCH color space for modern color handling
- Use `cn()` from `@/lib/utils` to merge Tailwind classes

## Adding shadcn/ui Components

```bash
npx shadcn-ui@latest add [component-name]
```

Components will be added to `components/ui/` with the new-york style preset.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intenet1001-commits) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

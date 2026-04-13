## groton

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production with Turbopack  
- `npm start` - Start production server

## Architecture

This is a Next.js 15 project using the App Router architecture with TypeScript and Tailwind CSS.

### Tech Stack
- **Framework**: Next.js 15 with App Router
- **Styling**: Tailwind CSS v4 with shadcn/ui components
- **UI Components**: Headless UI, Heroicons, Lucide React
- **Animation**: Motion library
- **Build**: Turbopack (Next.js's faster bundler)

### Project Structure
- `/src/app/` - App Router pages and layouts
- `/src/components/` - Reusable React components including shadcn/ui components
- `/src/lib/` - Utility functions and shared logic
- `/public/` - Static assets

### Component System
- Uses shadcn/ui with "new-york" style variant
- Components are configured via `components.json`
- Utility function `cn()` from `/src/lib/utils.ts` combines clsx and tailwind-merge
- Path aliases configured: `@/` maps to `./src/`

### Styling
- Tailwind CSS v4 with PostCSS
- CSS variables enabled for theming
- Global styles in `/src/app/globals.css`
- Geist font family (sans and mono) loaded via next/font

### Key Files
- `components.json` - shadcn/ui configuration
- `next.config.ts` - Next.js configuration (currently minimal)
- Path aliases configured in both `tsconfig.json` and `components.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexdocta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexdocta)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## nextjs-atomic-design-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server with Turbopack (opens on http://localhost:3000)
- `npm run build` - Build production bundle with Turbopack
- `npm start` - Start production server

## Architecture Overview

This is a Next.js 15 application using the App Router architecture with atomic design principles. The project is set up with modern tooling including Turbopack for faster builds and development.

### Key Technologies
- **Next.js 15** with App Router and React 19
- **TypeScript** for type safety
- **Tailwind CSS v4** for styling with PostCSS
- **shadcn/ui** component library (New York style variant)
- **Turbopack** for optimized builds and development

### Project Structure
- `src/app/` - Next.js App Router pages and layouts
- `src/lib/` - Shared utilities and helpers
- `components.json` - shadcn/ui configuration with path aliases

### Component Strategy
The project follows atomic design principles and is configured for shadcn/ui components:
- Components are aliased to `@/components`
- UI components go in `@/components/ui`
- Utilities are in `@/lib` (aliased as `@/lib/utils`)
- Uses Lucide React for icons
- CSS variables enabled for theming

### Path Aliases
- `@/components` → `src/components`
- `@/lib` → `src/lib`
- `@/utils` → `src/lib/utils`
- `@/ui` → `src/components/ui`
- `@/hooks` → `src/hooks`

### Styling
- Tailwind CSS v4 with neutral base color
- CSS variables enabled for dynamic theming
- `cn()` utility function combines clsx and tailwind-merge for conditional classes
- Geist font family loaded via next/font

---
> Source: [p-t-a-p-1/nextjs-atomic-design-demo](https://github.com/p-t-a-p-1/nextjs-atomic-design-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->

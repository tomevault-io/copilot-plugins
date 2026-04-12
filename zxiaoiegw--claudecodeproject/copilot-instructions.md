## claudecodeproject

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation First

**IMPORTANT:** Before generating any code, ALWAYS check the `/docs` directory for relevant documentation files. These docs contain project-specific patterns, conventions, and implementation details that must be followed. Read and apply the guidance from any relevant docs before writing or modifying code.

- /docs/ui.md
- /docs/data-fetching.md

## Commands

- `npm run dev` - Start dev server (localhost:3000)
- `npm run build` - Production build
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

No test framework is configured.

## Architecture

Next.js 16 app using the **App Router** with TypeScript strict mode, React 19, and Tailwind CSS v4.

- `app/` - App Router pages and layouts (React Server Components by default)
- `@/*` path alias maps to project root for imports
- Styling uses Tailwind CSS v4 via `@tailwindcss/postcss` with CSS variables for theming (dark mode via `prefers-color-scheme`)
- ESLint 9 flat config extending `next/core-web-vitals` and `next/typescript`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zxiaoiegw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zxiaoiegw)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

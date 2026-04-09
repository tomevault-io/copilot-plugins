## movement-metric

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Docs

Before generating any code, **always** read the relevant file(s) in the `/docs` directory first and strictly follow the standards defined there. The `/docs` directory is the source of truth for coding standards in this project.

- /docs/ui.md
- /docs/data-fetching.md
- /docs/auth.md
- /docs/data-mutations.md
- /docs/routing.md

## Commands

```bash
npm run dev      # Start development server
npm run build    # Production build
npm run start    # Start production server
npm run lint     # Run ESLint
```

## Architecture

This is a **Next.js 16 App Router** project using TypeScript and Tailwind CSS v4.

- **Routing**: File-based via `/app` directory (App Router, not Pages Router)
- **Styling**: Tailwind CSS v4 via PostCSS; global styles and CSS theme variables in `app/globals.css`
- **Fonts**: Geist Sans/Mono loaded via `next/font` in `app/layout.tsx`
- **Path alias**: `@/*` maps to the project root

**Stack**: Next.js 16, React 19, TypeScript 5 (strict mode), Tailwind CSS 4, ESLint 9

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samuellaudev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Samuellaudev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

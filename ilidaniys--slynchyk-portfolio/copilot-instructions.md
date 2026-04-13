## slynchyk-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Slynchyk Andreu, built with the T3 Stack (create-t3-app). Single-page application displaying work history, education, skills, and projects.

## Commands

- `npm run dev` — Start dev server with Turbopack
- `npm run build` — Production build
- `npm run check` — Run both lint and typecheck
- `npm run lint` — ESLint only
- `npm run lint:fix` — ESLint with auto-fix
- `npm run typecheck` — TypeScript type checking (`tsc --noEmit`)
- `npm run format:write` — Format with Prettier
- `npm run format:check` — Check formatting
- `SKIP_ENV_VALIDATION=1 npm run build` — Build without env validation (useful for CI/Docker)

## Architecture

**Stack:** Next.js 15 (App Router) + React 18 + TypeScript + Tailwind CSS 3

**Single-page layout** — All content renders in `src/app/page.tsx` via section components:
`MainWrapper` → `Header` → `WorkStudySection` → `Skills` → `ProjectSection` → `Footer`

**Route groups as section organizers** — Parenthesized directories (`(headerPart)`, `(history)`, `(skills)`, `(projects)`, `(footer)`) group related components without creating URL routes.

**UI components** — Uses shadcn/ui (new-york style) with Radix primitives. Components live in `src/components/ui/`. Add new shadcn components via the CLI: `npx shadcn@latest add <component>`.

**Path alias** — `~/` maps to `./src/` (configured in `tsconfig.json`).

**Environment validation** — `@t3-oss/env-nextjs` with Zod schemas in `src/env.js`. Add new env vars there and in `.env.example`.

**Styling** — Dark background (#181922) with white text. CSS variables for theming defined in `src/styles/globals.css`. Custom Tailwind animations: `ripple`, `popUp`, `wiggle`, `fill` (defined in `tailwind.config.ts`). Uses `tailwind-scrollbar` and `tailwindcss-animate` plugins.

## Code Conventions

- Use `type` imports (`import { type Foo }`) — enforced by ESLint
- Prefix unused args with `_` — ESLint warns on unused vars otherwise
- Font: Geist Sans (loaded in layout.tsx via CSS variable)
- Static images are imported from `public/` and used with Next.js `Image` component
- `cn()` utility from `src/lib/utils.ts` for merging Tailwind classes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ilidaniys)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ilidaniys)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

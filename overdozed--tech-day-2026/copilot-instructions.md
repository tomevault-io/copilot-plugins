## tech-day-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MoepFlix** is a one-day hackathon project to build a streaming platform with an embedded mini-game. During video loading (typically 10-20 seconds), users play a Tinder-style swipe quiz with AI-generated, video-relevant questions. Correct answers slow the loading bar; wrong answers speed it up. The tech stack prioritizes pragmatism over perfection—scope is ruthlessly reduced if implementation gets stuck.

The full implementation concept (architecture, component details, required packages) is documented in `docs/INITIAL_IMPLEMENTATION_CONCEPT.md`. Read this before making architectural decisions.

## Tech Stack & Key Constraints

- **Framework:** Next.js 16.2.0 (App Router)
- **Runtime:** React 19.2.4
- **Styling:** Tailwind CSS v4
- **TypeScript:** Strict mode enabled
- **Compiler:** React Compiler is enabled in `next.config.ts`
- **ESLint:** ESLint 9 (flat config format)

⚠️ **Critical:** Next.js 16 and React 19 have breaking changes from older versions. Before writing code, consult the Next.js docs in `node_modules/next/dist/docs/` and heed any deprecation notices. See `AGENTS.md` for details.

## Common Commands

```bash
# Development
npm run dev          # Start dev server (localhost:3000)

# Build & Production
npm run build        # Next.js build
npm run start        # Production server

# Code Quality
npm run lint         # ESLint check
```

## Project Structure

- **app/** — Next.js App Router root. Currently minimal (root `layout.tsx`, `page.tsx`). Expand with nested routes and layout groups as needed.
- **docs/** — `INITIAL_IMPLEMENTATION_CONCEPT.md` contains the full architecture design, component sketches, and implementation approach.
- **public/** — Static assets.
- **postcss.config.mjs** — Tailwind CSS v4 configuration.
- **tsconfig.json** — Path alias `@/*` maps to project root (use `@/app`, `@/components`, etc.).
- **eslint.config.mjs** — ESLint 9 flat config with Next.js rules.

## Key Implementation Details

### Component Architecture
The implementation concept document outlines:
- A **bento grid** features section (framer-motion-powered)
- A **swipe card** quiz component for the mini-game (framer-motion)
- Question **buffering system** to pre-load 3+ questions asynchronously
- AI-generated question queue tied to video content

### Required Dependencies
The concept doc specifies these packages (not yet installed):
- `lucide-react` — Icons
- `framer-motion` — Animations & swipe gestures
- `@remotion/player` — Animated question "videos"

Install these when beginning implementation.

### TypeScript & Strict Mode
- Strict mode is enabled; all code must be type-safe
- Use `@/*` path aliases when importing from the project root
- Next.js plugin is active in `tsconfig.json`

## Development Workflow

1. **Read `docs/INITIAL_IMPLEMENTATION_CONCEPT.md`** before starting new features to understand the intended architecture.
2. **Focus on App Router patterns:** Use Server Components by default, add Client Components only where necessary (interactivity, `useState`, event handlers).
3. **Tailwind CSS v4** uses the new `@import "tailwindcss"` syntax (see `app/globals.css`); do not use the old `@tailwind` directives.
4. **React Compiler enabled:** Rely on it for automatic memoization; you do not need to manually wrap components in `memo()`.

## Testing & Linting

No test framework is configured yet. When adding tests, consider the scope constraints (one-day hackathon) and set up minimal infrastructure (e.g., Vitest or Jest) only if truly necessary for stability.

Always run `npm run lint` before committing to catch ESLint violations early.

## Notes for Future Contributors

- **Scope is flexible.** If features become blockers, reduce scope ruthlessly rather than over-engineering solutions.
- **Prioritize user experience.** The swipe quiz and loading bar interaction are core; polish these first.
- **AI-first mindset.** The questions, images, and timing are dynamically generated. Design the UI to gracefully handle variable content.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Overdozed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

## ralph-watch

> - Next.js 14 (App Router) + TypeScript

# Ralph Watch - Project Config

## Tech Stack

- Next.js 14 (App Router) + TypeScript
- pnpm package manager
- Tailwind CSS + shadcn/ui components
- tRPC for API layer
- Zod for validation (use .passthrough() for loose schemas)
- chokidar for file watching

## Commands

- `pnpm dev` - start dev server
- `pnpm build` - production build
- `pnpm check` - lint + typecheck + tests (must pass before completing ticket)

## Directory Structure

- Components in src/components/
- tRPC routers in src/server/routers/
- Shared types/schemas in src/lib/
- Use RALPH_DIR env var for target directory (default: cwd)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jbarlo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

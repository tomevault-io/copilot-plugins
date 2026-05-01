## trivet

> - Next.js (App Router) + React 18 + TypeScript

# Trivet AGENTS

## Project basics
- Next.js (App Router) + React 18 + TypeScript
- Tailwind + shadcn/ui for UI
- Prisma + Postgres
- Package manager: pnpm

## Local commands
- Install: `pnpm install`
- Dev server: `pnpm dev`
- Build: `pnpm build`
- Typecheck: `pnpm check`
- Lint: `pnpm lint`
- Format: `pnpm format`
- Format check: `pnpm format:check`
- Prisma client: `pnpm prisma:generate`
- Migrations: `pnpm prisma:migrate`

## Repo rules of thumb
- Keep loading states as skeletons (no spinner-only loading).
- Align page content with the navbar container (`max-w-6xl px-6`).
- Match Contraption styling (Helvetica, rounded buttons, neutral palette).
- Use `.env` locally only; update `.env.example` when adding variables; production uses injected env vars.

## CI expectations
- `pnpm install`, `pnpm check`, `pnpm lint`, `pnpm format:check`, `pnpm build`, and Docker build must pass.

---
> Source: [contraptionco/trivet](https://github.com/contraptionco/trivet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->

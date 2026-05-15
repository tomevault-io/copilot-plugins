## odtok-novakova

> Water level monitoring dashboard (Next.js App Router, MUI + Tailwind, Recharts, TanStack Query).

# AGENTS.md

## Project

Water level monitoring dashboard (Next.js App Router, MUI + Tailwind, Recharts, TanStack Query).

## Tooling

**Runtime:** [mise-en-place](https://mise.jdx.dev) manages the `bun` version — run `mise install` before anything else to get the correct Bun version (1.3.12) on your PATH.

**Package manager:** `bun` (not npm/yarn/pnpm).

## Setup

```bash
mise install   # installs bun 1.3.12
bun install    # installs dependencies
```

## Stack

- **Framework:** Next.js (App Router)
- **UI:** MUI v9 + Tailwind CSS v4 + Emotion
- **Charts:** Recharts
- **Data fetching:** TanStack Query v5
- **Date handling:** date-fns v4
- **Language:** TypeScript (strict)

## Code conventions

- Tabs for indentation
- No comments — prefer self-documenting names
- Linter config in `eslint.config.mjs` is authoritative; defer to it

---
> Source: [petrmiko/odtok-novakova](https://github.com/petrmiko/odtok-novakova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->

## harness-bench

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Turborepo monorepo with a Next.js 16 web app and shared packages, using Bun as the package manager and shadcn/ui for components.

**Stack:** Next.js 16 (App Router), React 19, TypeScript 5.9, Tailwind CSS v4, Radix UI, CVA

## Commands

- `bun install` — install dependencies
- `bun run dev` — start all apps in dev mode (Next.js with Turbopack)
- `bun run build` — build all apps/packages
- `bun run lint` — lint all apps/packages
- `bun run format` — format with Prettier (`**/*.{ts,tsx,md}`)

### Per-workspace commands

```bash
bun run --cwd apps/web lint:fix    # Lint and auto-fix web app
bun run --cwd apps/web typecheck   # Type-check web app (tsc --noEmit)
bun run --cwd packages/ui lint     # Lint UI package (--max-warnings 0)
```

### Things to note

- Always assume a dev server is already running
- When working with bun, always use the `bun-package-manager` skill
- When working with react, always use the `vercel-react-best-practices` skill
- No test framework is configured yet

## Architecture

- **`apps/web`** — Next.js 16 app (App Router, Turbopack dev server)
- **`packages/ui`** — Shared UI component library (`@workspace/ui`), built with shadcn/ui + Radix UI + CVA
- **`packages/eslint-config`** — Shared ESLint flat configs (ESLint 9)
- **`packages/typescript-config`** — Shared TypeScript configs

## Key Conventions

- **Package manager:** Bun (`bun@1.3.8`) — never use npm/yarn/pnpm
- **Adding shadcn components:** `bunx shadcn@latest add <component> -c apps/web` (places components in `packages/ui/src/components/`)
- **Importing UI components:** `import { Button } from "@workspace/ui/components/button"`
- **UI utility function:** `cn()` from `@workspace/ui/lib/utils` (clsx + tailwind-merge)
- **Global styles:** `@workspace/ui/globals.css`
- **App-local imports:** `@/*` alias resolves to `apps/web/*`

## Code Style

- **No semicolons**, double quotes, 2-space indentation (Prettier defaults)
- **TypeScript strict mode** with `noUncheckedIndexedAccess: true`
- All packages use ESM (`"type": "module"`)
- Files: kebab-case (`button.tsx`). Components: PascalCase. Functions/vars: camelCase.
- Use `type` keyword for type-only imports (`import { type VariantProps }`)

### Component Patterns

- Default to React Server Components — only add `"use client"` when needed
- Use `function` declarations for exported components (not arrow functions)
- Use `data-slot` attributes on component root elements
- Use `React.ComponentProps<"element">` for extending HTML element props
- Tailwind CSS v4 configured via CSS (no `tailwind.config` file) — design tokens in `globals.css` `@theme inline` block
- Dark mode via `.dark` class (managed by `next-themes`)

### Import Order

1. React / framework imports (`react`, `next/*`)
2. External library imports
3. Workspace package imports (`@workspace/ui/*`)
4. Local alias imports (`@/*`)
5. Relative imports

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adiadd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

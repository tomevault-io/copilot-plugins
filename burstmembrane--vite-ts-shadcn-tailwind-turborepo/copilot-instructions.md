## vite-ts-shadcn-tailwind-turborepo

> This document defines the strict rules all AI agents and contributors must follow when modifying code in this monorepo.

# Cursor Rules

This document defines the strict rules all AI agents and contributors must follow when modifying code in this monorepo.
These requirements apply to all apps and packages.

---

## 1. Monorepo Overview

This repository is a **pnpm workspace + Turborepo monorepo**, consisting of:

```
apps/
  web/            # Main React 19 + Vite application
packages/
  ui/             # shadcn-based shared UI library
  lib/            # Shared utilities & internal modules
  schema/         # Shared Zod schemas (optional)
```

### Core Tooling

- **pnpm** as the package manager
- **pnpm workspaces** for dependency sharing
- **Turborepo** for build orchestration
- **Vite** for frontend builds
- **React 19** + React Compiler
- **TypeScript** strict mode
- **Tailwind CSS v4**
- **shadcn/ui** (New York style) but defined inside a package (`packages/ui`)

---

## 2. pnpm Workspaces & Package Naming Rules

### Workspace Root (pnpm-workspace.yaml)

All packages live in:

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

### Package Naming

Each package must have a scoped alias matching its directory:

```
packages/ui      →  @repo/ui
packages/lib     →  @repo/lib
packages/schema  →  @schema
packages/config  →  @config
```

These aliases must be used:

- in Vite config (`resolve.alias`)
- in TypeScript config (`paths`)
- in Turbo pipelines (per-package scripts)

### Import Rules (NON-NEGOTIABLE)

**Allowed:**

```typescript
import { Button } from "@repo/ui/button";
import { formatDate } from "@repo/lib/date";
import { userSchema } from "@schema/user";
```

**Forbidden:**

```typescript
import "../../../packages/ui/button";
import { x } from "packages/lib/date";
import { y } from "@/../../packages/schema";
```

**Relative imports MAY NOT cross package boundaries.**

---

## 3. TypeScript Rules (Strict Mode Enforcement)

The entire monorepo uses a single strict TS base config (`tsconfig.base.json`).

These rules must remain enabled:

- `strict: true`
- `noImplicitAny: true`
- `exactOptionalPropertyTypes: true`
- `noUncheckedIndexedAccess: true`
- `noImplicitReturns: true`
- `noImplicitOverride: true`
- `useUnknownInCatchVariables: true`
- `verbatimModuleSyntax: true`
- `moduleResolution: "bundler"`

### Prohibited:

- `any` (unless explicitly annotated with `// @ts-expect-error` and justification)
- default exports
- implicit any return types
- interface/prop widening
- leaking React-only types into packages that should remain framework-agnostic

**All packages must typecheck in isolation.**

---

## 4. ESLint Rules (Strict)

A unified root ESLint config governs all apps + packages, enforcing:

- `no-explicit-any: error`
- `consistent-type-imports: error`
- `import/no-default-export: error`
- `unicorn/filename-case` (camelCase, PascalCase, or kebab-case)
- no unused vars
- no unused imports
- no bare console logs except `warn` and `error`

**Any ESLint error invalidates the PR.**
Claude must not generate code that violates these rules.

---

## 5. Monorepo Architecture Rules

### 5.1 Apps (apps/\*)

Apps must contain:

- feature components
- routing
- screens/pages
- application state
- glue logic

Apps may import from packages, but **apps must not export anything for package consumption**.

### 5.2 Packages (packages/\*)

Packages hold all reusable logic:

#### packages/ui

- All shadcn/ui components live here
- Components must:
  - Use CVA for variants
  - Use Tailwind tokens
  - Use the shared `cn()` utility
- Must be consumable by any workspace app
- Must compile without app-specific context

#### packages/lib

- Shared logic, utilities, helpers
- Must avoid React unless explicitly designed as React utilities

#### packages/schema

- Zod schemas
- DTOs, validation layers
- Shared client/server contract definitions (if needed)

**Packages may not import from apps.**

---

## 6. Build Pipeline Rules (Turbo)

Turbo enforces the pipeline:

```
lint → typecheck → build
```

### Requirements:

- All packages must provide `lint`, `typecheck`, and `build` scripts
- No code may enter the repository that fails any part of the pipeline
- Claude must refuse to produce code that breaks the pipeline

### Example Turbo Pipeline

```json
{
  "pipeline": {
    "lint": { "outputs": [] },
    "typecheck": { "dependsOn": ["^typecheck"], "outputs": [] },
    "build": { "dependsOn": ["^build", "typecheck"], "outputs": ["dist/**"] },
    "dev": { "cache": false }
  }
}
```

---

## 7. React + Vite Requirements

- React 19 with **React Compiler enabled** (Claude must not disable it)
- No manual memoization (`useMemo`, `useCallback`) unless profiling proves needed
- Components must be pure unless justified
- Lazy-load heavy components
- No external state management libraries without explicit approval
- Vite config must preserve package alias resolution (`@repo/ui`, `@repo/lib`, etc.)

---

## 8. Tailwind CSS v4 Rules

- Tailwind must remain **CSS-first** (`@theme` in `index.css`)
- No alternative styling systems (styled-components, emotion, stitches, etc.)
- All design tokens must come from Tailwind or local CSS variables
- UI components must not inline arbitrary styles that break theming consistency

---

## 9. File Naming & Exports

- React components: `PascalCase.tsx`
- Utilities: `camelCase.ts` or `kebab-case.ts`
- Config files: `kebab-case.config.ts`
- **No default exports**
- Packages must expose entrypoints via `index.ts`
- No circular imports

### Example allowed:

```typescript
export function Button() {}
export type ButtonProps = { ... }
```

### Example forbidden:

```typescript
export default function Button() {}
```

---

## 10. Testing Rules

Tests are **optional** unless:

- logic is non-trivial
- the component interacts with user input
- the change modifies shared package logic

When required:

- Use **Vitest**
- Use **React Testing Library**
- Test behaviour, not implementation details

---

## 11. AI Agent Enforcement Rules

Claude (or any agent) must:

- Respect monorepo boundaries
- Never generate cross-package relative imports
- Always generate code that passes `pnpm lint`, `pnpm typecheck`, and `pnpm build`
- Use the correct package import aliases (`@repo/ui`, `@repo/lib`, etc.)
- Prefer creating new packages over adding shared logic to apps
- Refuse to introduce:
  - `any`
  - default exports
  - relaxed TS config
  - new dependencies without justification
- Ensure all output respects the Turbo pipeline

### If a user request violates these rules, Claude must:

1. Reject the request
2. Explain which rule would be violated
3. Suggest a compliant alternative

---

## Development Commands

All commands run from workspace root:

```bash
# Install dependencies
pnpm install

# Development server (all apps)
pnpm dev

# Build all packages and apps
pnpm build

# Lint all packages
pnpm lint

# Type check all packages
pnpm typecheck

# Run specific package command
pnpm --filter @repo/ui build
pnpm --filter web dev
```

---

## VERSION

**1.2.0** — Updated for pnpm, pnpm workspaces, Turbo monorepo, package-based import aliases.

**Last Updated:** 2025-12-09

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/burstMembrane) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

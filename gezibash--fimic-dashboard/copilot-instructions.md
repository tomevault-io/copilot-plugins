## fimic-dashboard

> bun dev                # Start development server with Turbopack

# Agent Development Guide - Fimic Dashboard

## Build & Development Commands
```bash
bun dev                # Start development server with Turbopack
bun build              # Build for production
bun lint               # Run Biome linter (ultracite wrapper)
bun tsc                # TypeScript type checking  
bun format             # Format code with Biome
bun convex             # Start Convex dev server
```

## Code Style & Conventions
- **Framework**: Next.js 15 (App Router), React 19, TypeScript (strict mode)
- **Imports**: Use `'use client'` for client components. Prefer absolute imports: `@/components`, `@/lib`, `@/hooks`
- **Components**: Export named components. Place in `/components/ui` for reusable UI, `/app/*/components` for page-specific
- **Types**: Import Convex types as `type { Doc } from '@/../convex/_generated/dataModel'`. Define props with explicit types
- **Styling**: Tailwind CSS with cn() utility for conditional classes. Use shadcn/ui components from `@/components/ui`
- **State**: Use Convex for backend state (queries/mutations). Client state with React hooks
- **Validation**: Zod schemas in `/lib/validations`. Phone: Swiss (+41) or Kosovo (+383) formats
- **Error Handling**: Return early with null checks. Show user-friendly error states in components
- **File Naming**: kebab-case for files, PascalCase for components, camelCase for functions/variables

---
> Source: [gezibash/fimic-dashboard](https://github.com/gezibash/fimic-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->

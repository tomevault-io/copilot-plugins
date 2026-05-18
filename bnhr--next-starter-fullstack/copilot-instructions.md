## next-starter-fullstack

> Essential guidance for AI agents. Keep this minimal - research shows verbose context files reduce agent effectiveness.

# AGENTS.md

Essential guidance for AI agents. Keep this minimal - research shows verbose context files reduce agent effectiveness.

## Project Overview

Next.js 16 fullstack starter with authentication (Better Auth), database (Drizzle ORM + PostgreSQL), and type-safe environment variables (@t3-oss/env-nextjs).

## Tech Stack

Check `package.json` for versions and installed packages. Key stack:

- Next.js (App Router), React, TypeScript
- PostgreSQL + Drizzle ORM
- Better Auth (email/password + username)
- Tailwind CSS v4 + Radix UI + shadcn/ui
- Package Manager: Bun (never use npm/pnpm)

## JavaScript Ecosystem Guidance

The JavaScript ecosystem evolves rapidly. When uncertain about APIs, package versions, or deprecations:

1. **Use web search/fetch** to verify current best practices
2. Check official documentation for breaking changes
3. Verify package compatibility before suggesting updates
4. Look for deprecation warnings in recent releases

Don't assume knowledge is current - verify when in doubt.

## Commands

```bash
bun dev          # Start dev server
bun run build    # Production build
bun run start    # Start production server
bun run lint     # Run ESLint
bun run db:migrate    # Apply DB migrations
bun run db:generate   # Generate new migration
bun run db:studio     # Open Drizzle Studio
```

## Package Management

Always use `bun` for package operations (install, add, remove, update). Never use `npm` or `pnpm`.

```bash
bun install              # Install all dependencies
bun add <package>        # Add a dependency
bun add -D <package>     # Add a dev dependency
bun remove <package>     # Remove a dependency
bun update <package>     # Update a dependency
```

## Key Files

| File             | Purpose                                              |
| ---------------- | ---------------------------------------------------- |
| `lib/env.ts`     | Type-safe env vars (use `env.VAR` not `process.env`) |
| `db/schema.ts`   | Drizzle schema definitions                           |
| `lib/auth.ts`    | Better Auth configuration                            |
| `features/auth/` | Login/Register forms with Valibot schemas            |

## Architecture

Feature-based organization: `features/{feature}/` contains components, schemas, utils. Server actions in `app/actions/{feature}.ts`. Generic UI in `components/ui/`.

## Code Conventions

- Use `@/` alias for absolute imports
- Always use `env.VAR` from `@/lib/env`, never `process.env`
- Forms use Valibot schemas in `features/{feature}/schema.ts`
- Prefer server actions over API routes
- Use shadcn CLI for UI components: `bunx --bun shadcn@latest add <component>`

## Database Changes

1. Update `db/schema.ts`
2. Run `npx drizzle-kit generate`
3. Run `npx drizzle-kit migrate`

## Adding Features

1. Create `features/{feature}/` with components and schema
2. Add server actions in `app/actions/{feature}.ts`
3. Add routes in `app/{feature}/`
4. Apply database changes if needed (see above)
5. Use shadcn CLI for generic UI components

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->

---
> Source: [bnhr/next-starter-fullstack](https://github.com/bnhr/next-starter-fullstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

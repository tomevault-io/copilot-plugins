## xpecialist-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
bun --bun run dev          # Start dev server on port 3000
npx convex dev             # Start Convex backend (run in separate terminal)

# Build & Test
bun --bun run build        # Production build
bun --bun run test         # Run all tests
npx vitest run src/path/to/file.test.ts  # Run single test file

# Code Quality
bun --bun run check        # Biome lint + format check
bun --bun run lint         # Lint only
bun --bun run format       # Format only

# Add UI Components
pnpm dlx shadcn@latest add <component>
```

## Architecture

**Stack:** TanStack Start (React 19) + Convex + Clerk + Tailwind CSS v4

**Provider Hierarchy** (`src/routes/__root.tsx`):
```
ConvexProvider → ClerkProvider → [App Content]
```

**Routing:** File-based routing via TanStack Router
- Routes in `src/routes/`
- Layout in `src/routes/__root.tsx`
- Demo routes (prefixed with `demo/`) can be deleted

**Backend:** Convex (serverless database + functions)
- Schema: `convex/schema.ts`
- Functions: `convex/*.ts` (queries/mutations)
- Use `v` from `convex/values` for validators
- System fields `_id` and `_creationTime` are auto-generated

**Integrations** (`src/integrations/`):
- `convex/provider.tsx` - Convex client setup with React Query
- `clerk/provider.tsx` - Auth provider
- `tanstack-query/` - Query client + devtools

## Convex Patterns

Schema validators use `v` builder:
```ts
v.string(), v.number(), v.boolean(), v.id("tableName")
v.optional(v.string()), v.union(v.literal("a"), v.literal("b"))
v.object({ field: v.string() }), v.array(v.string())
```

Query/mutation pattern:
```ts
export const myQuery = query({
  args: { id: v.id('tableName') },
  handler: async (ctx, args) => {
    return await ctx.db.get(args.id)
  },
})
```

## Environment Variables

Define in `src/env.ts` using T3Env + Zod. Client vars must use `VITE_` prefix.

Required in `.env.local`:
- `VITE_CONVEX_URL` - Convex deployment URL
- `CONVEX_DEPLOYMENT` - Convex deployment name
- `VITE_CLERK_PUBLISHABLE_KEY` - Clerk auth key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/williamneves) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

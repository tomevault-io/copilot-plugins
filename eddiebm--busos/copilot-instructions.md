## busos

> Core coding standards and non-negotiable rules for all work in the BUSOS / FounderPath codebase. Always apply these rules.


# BUSOS Coding Standards

These rules apply to every file in the codebase without exception.

## Design System — Semantic Tokens Only

Never use raw Tailwind palette classes. Always use semantic tokens.

| Wrong | Correct |
|---|---|
| `bg-white` | `bg-background` |
| `text-zinc-900` | `text-foreground` |
| `bg-zinc-800` | `bg-card` |
| `text-zinc-400` | `text-muted-foreground` |
| `border-zinc-700` | `border-border` |
| `bg-violet-600` | `bg-primary` |
| `text-violet-400` | `text-primary` |
| `bg-red-500` | `bg-destructive` |

The full token set is defined in `apps/web/app/globals.css`.

## Authentication — Every API Route

Every API route handler must authenticate the user and verify venture access before touching the database:

```typescript
const userId = await getOrCreateUserFromClerk();
if (!userId) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });

const access = await getVentureAccess(ventureId, userId);
if (!access) return NextResponse.json({ error: "Not found" }, { status: 404 });

// For write operations:
if (!canWrite(access.role)) return NextResponse.json({ error: "Forbidden" }, { status: 403 });
```

## TypeScript — No `any`

All data shapes must be explicitly typed. Use Prisma-generated types where possible. For API response shapes, define a local `type` or `interface`.

## Prisma — Schema Changes

Whenever `schema.prisma` is modified:
1. Run `npx prisma generate` immediately.
2. If adding new fields, create a migration: `npx prisma migrate dev --name describe-the-change`.
3. Never modify the schema without running generate — TypeScript will break.

## Build Gate

Before declaring any work complete, the build must pass:

```bash
cd /home/ubuntu/busos && pnpm --filter @busos/web build
```

Zero TypeScript errors is the only acceptable outcome.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Eddiebm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Eddiebm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

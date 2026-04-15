## banking

> > **Project:** Next.js 16, TypeScript (strict), Drizzle ORM,

# Banking Project - Cursor AI Agent Rules

> **Project:** Next.js 16, TypeScript (strict), Drizzle ORM,
> PostgreSQL, NextAuth, shadcn/UI, Tailwind CSS v4
>
> **Last Updated:** 2026-04-05

---

## Source of Truth

This file is a quick reference for Cursor AI. For detailed patterns, see:

- **[AGENTS.md](./AGENTS.md)** — Stack, PR-blocking rules, commands,
  core patterns (Reference + How-to)
- **[`.cursor/rules/`](.cursor/rules/)** — Enforced workspace rules
  (coding standards, Server Actions, DAL, etc.)

### Sync Checklist

When updating this file, verify alignment with:

- `AGENTS.md` — commands, rules, project structure
- `.github/copilot-instructions.md` — overlapping content should match
- package.json — scripts must match actual commands

---

## Critical Rules (PR Blocking)

| Rule | Requirement |
| --- | --- |
| No `any` types | Use `unknown` + type guards |
| No N+1 queries | Eager load / JOIN — no per-row queries in loops |
| No raw `process.env` | Use `lib/env.ts` with Zod validation |
| All mutations via Server Actions | Never API routes for mutations |
| Zero TypeScript errors | Pass `npm run type-check` |
| Zero lint warnings | Pass `npm run lint:strict` |
| All tests pass | Pass `npm run test` |

---

## Quick Reference

### Server Action Pattern

```typescript
"use server";

import { auth } from "@/lib/auth";
import { revalidatePath } from "next/cache";
import { z } from "zod";

export async function actionName(input: unknown) {
  const session = await auth();
  if (!session?.user?.id) return { ok: false, error: "Not authenticated" };

  const parsed = Schema.safeParse(input);
  if (!parsed.success) return { ok: false, error: "Invalid input" };

  try {
    await dal.operation(parsed.data);
    revalidatePath("/");
    return { ok: true };
  } catch {
    return { ok: false, error: "Operation failed" };
  }
}
```

### DAL Pattern (JOIN for relations)

```typescript
// GOOD - single query with JOIN
export class UserDal {
  async findByIdWithProfile(id: string) {
    const [result] = await db
      .select({ ... })
      .from(users)
      .leftJoin(user_profiles, eq(users.id, user_profiles.userId))
      .where(eq(users.id, id));
    return result;
  }
}
export const userDal = new UserDal();
```

### Auth Pattern

```typescript
import { auth } from "@/lib/auth";

export default async function Page() {
  const session = await auth();
  if (!session?.user?.id) redirect("/sign-in");
  // Use session.user.id...
}
```

---

## Available Commands

```bash
# Validation (required before commit)
npm run validate        # Run all checks (format, type-check, lint, test)
npm run lint:strict     # ESLint (blocks PR at 0 warnings)
npm run type-check      # TypeScript type checking

# Development
npm run dev            # Dev server (localhost:3000)
npm run build          # Production build

# Database
npm run db:studio      # Drizzle Studio
npm run db:push        # Push schema changes to DB
npm run db:migrate     # Run migrations
npm run db:generate    # Generate migrations
npm run db:check      # Drizzle migration/schema check

# Testing
npm run test           # All tests (Vitest + Playwright)
npm exec vitest run tests/unit/register.test.ts --config=vitest.config.ts
npm exec playwright test tests/e2e/sign-in.spec.ts
```

---

## Project Structure

```text
app/
├── (auth)/           # sign-in, sign-up
├── (root)/           # dashboard, protected routes
├── api/              # API routes (minimal)
components/ui/        # shadcn components
lib/
├── actions/          # Server Actions (*.actions.ts)
├── dal/              # Data Access Layer (*.dal.ts)
├── auth.ts           # auth() helper
├── env.ts            # Zod env validation
└── encryption.ts     # AES-256-GCM
database/
├── db.ts             # DB connection
└── schema.ts         # Drizzle schema
tests/
├── unit/             # Vitest tests
└── e2e/              # Playwright tests
```

---

## Key Files

| File | Purpose |
| --- | --- |
| `lib/auth.ts` | Authentication helper |
| `lib/dal/user.dal.ts` | User data access |
| `lib/env.ts` | Environment variable validation |
| `database/schema.ts` | Database schema definition |
| `next.config.ts` | React Compiler enabled (`reactCompiler: true`) |

---

**End of .cursorrules** — See [AGENTS.md](./AGENTS.md) and
[`.cursor/rules/`](.cursor/rules/) for full documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rhixecompany) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## pulumi-dashboard

> This project follows the `node-typescript` skillset. Apply these rules consistently to all code you write or modify.

# Pulumi Dashboard — Claude Code Instructions

## Skill: node-typescript

This project follows the `node-typescript` skillset. Apply these rules consistently to all code you write or modify.

---

## Git

- Split commits by logical concern — one commit per layer (deps, tooling, auth, data, UI)
- Do not add `Co-Authored-By` trailers to commit messages

---

## Formatting & linting

Tooling: **Biome v2** (`biome.json`).

- Run `npm run check` after making changes — it formats, lints, and sorts imports in one pass
- Run `npm run depcheck` to check for unused/missing dependencies (knip)
- Run `npm run typecheck` for TypeScript validation

Never bypass these tools. If a rule fires, fix the root cause rather than adding a `biome-ignore` comment.

### Style rules enforced by Biome

| Rule | Value |
|---|---|
| Quotes | Single (`'`) |
| Semicolons | None (omit trailing `;`) |
| Indent | 2 spaces |
| Line width | 100 chars |
| Trailing commas | All |
| Import sorting | Auto-sorted by Biome assist |

---

## Function style

Use **named function declarations** for all top-level and exported functions. Arrow functions are only for inline callbacks.

```ts
// ✅ correct — named declaration
export async function listStacks(): Promise<StackSummary[]> { ... }

export default function StacksPage() {
  return <div />
}

// ✅ correct — arrow only for inline callbacks
const sorted = items.sort((a, b) => a.name.localeCompare(b.name))
items.forEach((item) => console.log(item))

// ❌ wrong — arrow for top-level
export const listStacks = async (): Promise<StackSummary[]> => { ... }
const StacksPage = () => <div />
```

---

## TypeScript

- Prefer `interface` over `type` for object shapes
- Use `satisfies` when narrowing object literals against a type without widening
- Validate required env vars with an explicit `if (!VAR) throw new Error(...)` at module load — never use `!` assertions on `process.env`
- Mark any module that must never run on the client with `import 'server-only'` at the top

---

## Error handling

- Throw meaningful `Error` objects with context (e.g. `throw new Error(\`Empty body for key: ${key}\`)`)
- Do not swallow errors silently; let them propagate unless you have a deliberate fallback

---

## File & directory conventions

```
src/
  app/
    (dashboard)/          # Protected route group
    api/                  # API routes
    login/                # Public page
  lib/                    # Server-only utilities (s3.ts, pulumi-types.ts)
  components/ui/          # shadcn/ui components — do not edit generated files
  auth.ts                 # Auth.js config
  middleware.ts           # Route protection
```

- Place all server-only data-access code in `src/lib/`
- Do not create utility files for one-off operations
- Do not add barrel `index.ts` files

---

## Dependencies

- Pin all dependencies to exact versions — no `^`, `~`, or `*` ranges
- When adding a package, install it and copy the exact resolved version from `package-lock.json` into `package.json`
- After adding or removing a package, run `npm run depcheck` to confirm knip reports no issues

---
> Source: [koneru9999/pulumi-dashboard](https://github.com/koneru9999/pulumi-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->

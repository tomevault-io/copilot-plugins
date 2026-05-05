## typescript-strict-patterns

> TypeScript strict mode patterns to prevent type errors during build


# TypeScript Strict Mode (Essentials)

Keep these in mind to avoid “works in dev, fails in build” TypeScript errors.

## Common strict-mode traps

- **`Record<K, V>` defaults**: don’t use `{}` for required records. Prefer `createEmptyTeamRecord(...)` / `createEmptyTeamRecordFactory(...)`.
- **Nullable types**: never use non-null assertions (`!`). Use guard clauses or a safe fallback.
- **Literal inference**: avoid `as const` on objects you intend to reassign; annotate the type instead.
- **Prop types**: define shared aliases for “extended” types and reuse across parent/child.
- **Supabase + `Promise.all()`**: query builders aren’t `Promise`; use `PromiseLike<any>[]`.

```typescript
// from '@/lib/utils/types'
// Record defaults (strict-safe)
const teamMap = createEmptyTeamRecord<string[]>([])
const teamBuckets = createEmptyTeamRecordFactory<string[]>(() => [])

// Supabase Promise.all typing
const promises: PromiseLike<any>[] = []
promises.push(supabase.from('table').upsert(data))
await Promise.all(promises)
```

## Build Verification

Always run `npm run build` before committing to catch strict mode errors:

```bash
npm run build
```

The build performs full type checking that `npm run dev` may skip for performance.

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

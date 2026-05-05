## database-types

> Supabase/DB write-read type safety essentials (enums, UUID[] fields, DECIMAL FTE). Applies only to DB-facing code.


# Database / Supabase Type Safety (Essentials)

TypeScript types are **wider** than DB enums/columns. Prefer using conversion helpers from `@/lib/db/types` whenever writing/reading Supabase rows.

## Writing rows (must-do)

Use the preparation helpers whenever possible (`prepareTherapistAllocationForDb`, `preparePCAAllocationForDb`):

```typescript
await supabase
  .from('schedule_therapist_allocations')
  .upsert(prepareTherapistAllocationForDb({ allocation: therapistAlloc, specialPrograms }))

// same idea for PCAs:
await supabase
  .from('schedule_pca_allocations')
  .upsert(preparePCAAllocationForDb({ allocation: pcaAlloc, specialPrograms }))
```

If you’re building payloads manually, **always**:

- **`special_program_ids` (UUID[])**: `programNamesToIds(...)` then `assertValidSpecialProgramIds(...)`
- **`leave_type` (DB enum)**: `toDbLeaveType(tsLeaveType)`; store custom strings in `manual_override_note`
- **DECIMAL FTE**: run `normalizeFTE(...)` before writing any `fte_*` column

## Reading rows (must-do)

```typescript
const leaveType = fromDbLeaveType(
  dbAllocation.leave_type,
  dbAllocation.fte_remaining,
  dbAllocation.manual_override_note
)
```

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

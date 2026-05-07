## database

> Database schema and Supabase patterns


# Database Schema Rules

## Change Protocol (All 3 Steps Required)

1. `supabase/schema.sql` ← source of truth, bump version
2. `src/lib/schemaVersion.ts` ← sync EXPECTED_SCHEMA_VERSION
3. `src/types/supabase.ts` ← regenerate with `npm run gen:types`

## Supabase Query Patterns

```typescript
// GOOD: explicit columns, error handling
const { data, error } = await supabase
  .from('files')
  .select('id, name, status')
  .range(0, 49);
if (error) throw error;

// BAD: select(*), no error check
const { data } = await supabase.from('files').select('*');
```

## Storage

- Private files: signed URLs only
- Public assets: direct URLs OK

## Migration Naming

`YYYYMMDD-description.sql` → e.g., `20240115-add-workflow-status.sql`

## Schema Change Workflow

When making schema changes:
- **Only** update the schema files (`supabase/core.sql`, `supabase/modules/*.sql`)
- **Never** provide migration SQL in code blocks — user runs the full schema file
- **Never** run terminal commands to push SQL to Supabase
- Just update the files and confirm it's done

---
> Source: [bluerobotics/bluePLM](https://github.com/bluerobotics/bluePLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->

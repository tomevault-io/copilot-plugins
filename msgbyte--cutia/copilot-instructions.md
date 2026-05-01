## storage-migration

> Enforce storage migration scripts when project save format changes


# Storage Migration

When a code change modifies the persisted project data shape (fields in `TProject`, `TScene`, `TProjectMetadata`, `TProjectSettings`, `TimelineTrack`, or `TimelineElement` types), you **must** also implement a storage migration so existing saved projects upgrade automatically.

## When a Migration is Required

- Adding, removing, or renaming a field on any persisted type
- Changing the semantics or default of an existing persisted field
- Restructuring nested data (e.g. moving a field from root to `metadata`)

## Checklist

1. **Bump version** — increment `CURRENT_PROJECT_VERSION` in `services/storage/migrations/index.ts`
2. **Transformer** — create `transformers/vN-to-vM.ts` with pure transform logic
3. **Migration class** — create `vN-to-vM.ts` extending `StorageMigration` (`from = N`, `to = M`)
4. **Register** — add the new migration instance to the `migrations` array in `index.ts`
5. **Tests** — add `__tests__/vN-to-vM.test.ts` with fixture data covering normal, edge-case, and already-migrated scenarios

## File Structure

```
services/storage/migrations/
├── index.ts                        # CURRENT_PROJECT_VERSION + migrations array
├── base.ts                         # StorageMigration abstract class
├── runner.ts                       # Migration executor
├── vN-to-vM.ts                     # Migration class
├── transformers/
│   ├── vN-to-vM.ts                 # Pure transform function
│   ├── types.ts                    # ProjectRecord, MigrationResult
│   └── utils.ts                    # getProjectId, isRecord, etc.
└── __tests__/
    ├── vN-to-vM.test.ts
    └── fixtures/
        └── vN.ts                   # Fixture data for version N
```

## Migration Pattern

```typescript
// transformers/vN-to-vM.ts
export function transformProjectVNToVM({
  project,
}: {
  project: ProjectRecord;
}): MigrationResult<ProjectRecord> {
  const projectId = getProjectId({ project });
  if (!projectId) {
    return { project, skipped: true, reason: "no project id" };
  }

  if (isAlreadyMigrated({ project })) {
    return { project, skipped: true, reason: "already vM" };
  }

  const migratedProject = {
    ...project,
    /* apply changes */
    version: M,
  };

  return { project: migratedProject, skipped: false };
}
```

## Key Rules

- Transformers must be **pure functions** — no side-effects, no DB access
- Always guard with an `isAlreadyMigrated` check so re-runs are safe
- Never delete data without first copying it to the new location
- Keep each migration small and single-purpose

---
> Source: [msgbyte/cutia](https://github.com/msgbyte/cutia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->

## document-persistence

> Requirements when changing parameter set persistence (localStorage documents)


# Parameter set persistence

Parameter sets are the app **document**. Persisted shape is defined by `toSnapshot` + `schema.ts`, not `ExplorerState` alone.

**Read first:** [fe/src/lib/documents/README.md](fe/src/lib/documents/README.md)

## Rules

- **Never** load raw localStorage JSON via `Object.assign` or `mergeSnapshot` into live state. Use `parseSnapshot`.
- **Always** update `toSnapshot` and `schema.ts` coerce together when adding/removing persisted fields.
- **Always** follow the README decision tree (playbooks A–D) before finishing persistence work.
- Bump `CURRENT_SNAPSHOT_VERSION` and add `migrateVNToVN+1` only for breaking renames/format changes (playbook C).
- Add a `parse.test.ts` fixture when bumping schema version or changing load behavior.
- Run `npm run check` and `npm run test` after persistence changes.

## PR checklist

- [ ] Change type identified (A / B / C / D)
- [ ] `toSnapshot` and `schema.ts` in sync
- [ ] No raw JSON merge on load path
- [ ] Schema version bumped only if required
- [ ] Migration + test fixture if version bumped
- [ ] `migrate.ts` changelog updated
- [ ] `npm run check` and `npm run test` pass

---
> Source: [saabi/colorlab](https://github.com/saabi/colorlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->

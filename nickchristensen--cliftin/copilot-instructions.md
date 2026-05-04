## cliftin

> - Stack: `oclif` + TypeScript + `kysely` + `better-sqlite3`

# AGENTS.md

## Project
- Name: `cliftin`
- Stack: `oclif` + TypeScript + `kysely` + `better-sqlite3`
- Purpose: inspect/query Liftin workout SQLite data from CLI

## Runtime and Environment
- Node: target `>=24` (current dev setup uses `fnm`; keep project version files current).
- DB path must come from env var only: `LIFTIN_DB_PATH`.
- Local development expects `.env.local` with local copy of database.
- Missing/invalid DB path should fail fast with actionable error text.

## CLI Surface (Current)
- Top-level domains:
  - `programs`
  - `workouts`
  - `exercises`
- JSON mode: use `enableJsonFlag = true` for command JSON behavior.

## Data/Schema Conventions
- User-facing term is `week` (DB table is `ZPERIOD`).
- Active program resolution:
  - Prefer `ZWORKOUTPROGRAMSINFO.ZSELECTEDWORKOUTPROGRAMID` mapped to `ZWORKOUTPLAN.ZID`.
  - Do not rely solely on `ZWORKOUTPLAN.ZISCURRENT`.
- Ordering quirks:
  - Weeks: order by `ZPERIOD.Z_FOK_WORKOUTPLAN`, then `Z_PK`.
  - Routines within week: order by `ZROUTINE.Z_FOK_PERIOD`, then `Z_PK`.
- Planned RPE sentinel:
  - `16` means unspecified/default and should be normalized to `null`.
- Soft deletes:
  - Programs: hide soft-deleted rows (`ZSOFTDELETED = 1`) from list/detail output.

## Units and Weight Display
- Stored weight values appear to be kg-based in DB.
- Display units resolve from settings/equipment context:
  - Prefer `ZSETTINGS.ZMEASURMENTUNIT`.
  - Fallback to equipment metadata (`ZEQUIPMENT2.ZMEASURMENTUNIT`) where applicable.
- Imperial conversion rule (current): `kg * 2.2`.
- Human/table output:
  - show weight with suffix, e.g. `225 lb`.
- JSON output:
  - weight-like fields should be structured with units, e.g. `{ "value": 225, "unit": "lb" }`.
- Keep exercise snapshot + history row schema aligned via shared projector/serializer.

## Output and UX Conventions
- Keep `id` column first in tables.
- `tty-table` is used for table rendering.
- Value formatter styles numbers/booleans/dates; date parsing/formatting via `date-fns`.

## Quality Bar
- Before committing:
  - run `npm test`
  - ensure lint passes (posttest also runs lint)
- Prefer focused commits with descriptive messages.
- Document new DB quirks in `reference/db_schema.md`.

## Notes
- `ZWORKOUTPLAN.ZISTEMPLATE` exists but currently all rows are `0` in this dataset.
- Reference schema + quirks live in `reference/db_schema.md`.

---
> Source: [NickChristensen/cliftin](https://github.com/NickChristensen/cliftin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->

## chatxiv

> Always use ISO 8601 TEXT timestamps in SQLite schemas


# SQLite Timestamps: ISO 8601 Only

All timestamp columns in SQLite **must** use `TEXT NOT NULL` with ISO 8601 UTC strings (`YYYY-MM-DDTHH:MM:SS.sssZ`).

## Schema

```sql
created_at TEXT NOT NULL
```

Never use `INTEGER` for timestamps. ISO 8601 TEXT is human-readable in DB browsers and lexicographically sortable.

## TypeScript

- Column type: `string` (never `number`)
- Write: `new Date().toISOString()`
- Write future: `new Date(Date.now() + offsetMs).toISOString()`
- Compare: ISO 8601 UTC strings compare correctly with `<`, `>`, `=` in both JS and SQL

## SQL comparisons

```sql
-- Correct: compare TEXT to TEXT
WHERE expires_at < ?   -- bind: new Date().toISOString()

-- Wrong: never use epoch milliseconds
WHERE expires_at < ?   -- bind: Date.now()
```

## Rationale

Migration `004_timestamps_to_iso8601.sql` converted all legacy INTEGER epoch-ms columns. New tables must follow this convention from the start.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FF14-ChatBot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## 55-sync-cdc

> CDC pipeline conventions — ingest, event store, materialization, backfill, and gotchas


# CDC Pipeline Conventions

## Data Flow

```
Webhook → ingest.ts (normalize + store) → event-store.ts → Inngest cdc/materialize
→ consumer.ts (resolve adapter → ensureLiveTable → applyEvents) → destination DB
```

Backfills follow a parallel path: `backfill.ts` → `flow.execute` with `backfill: true` → adapter `applyBatch`.

## Key Files

| File | Responsibility |
|------|---------------|
| `ingest.ts` | Normalize events, append to event store, update sync state, enqueue materialization |
| `consumer.ts` | Read pending events, resolve destination adapter, materialize or drop |
| `backfill.ts` | Manage backfill runs, clear event store, trigger flow re-execution |
| `event-store.ts` | Append/read/mark events; idempotency via `stableStringify` + SHA1 |
| `normalization.ts` | Timestamp resolution, table naming, payload sanitization, dedup |
| `events.ts` | `CdcStoredEvent` type, `normalizeCdcEvent` |
| `sync-state.ts` | Entity-level state machine (stream transitions, consumer cursors) |
| `adapters/registry.ts` | `CdcDestinationAdapter` interface, `resolveCdcDestinationAdapter` |

## Rules

- **Idempotency** — event store keys use `stableStringify` + SHA1 for backfill records when no explicit `changeId`. Never modify the hashing inputs without understanding dedup impact.
- **Backfill soft-delete safety** — `consumer.ts` automatically softens hard-delete to soft-delete while `backfillState.status === "running"`. Do not bypass this check.
- **Timestamp resolution** — `normalization.ts` `resolveSourceTimestamp` mirrors `BaseConnector.resolveRecordTimestamp`. Changes to field precedence must be applied in both locations.
- **Consumer failure** — consumer throws after `markEventsFailed` + `applyStreamTransition(FAIL)`. Callers (Inngest functions) must expect retries.
- **Payload sanitization** — `sanitizeBackfillPayloadForIdempotency` strips `_mako_*` and `_syncedAt` fields before hashing. Adding new volatile fields requires updating this function.
- **Table naming** — `cdcLiveTableName` / `cdcStageTableName` use flow token from truncated ID + SHA1 fallback. Don't assume table names are human-readable.

## Destination Adapters

Adapters implement `CdcDestinationAdapter` and are manually registered in `resolveCdcDestinationAdapter`. See the `add-cdc-adapter` skill for scaffolding a new adapter.

Currently supported: BigQuery, PostgreSQL.

---
> Source: [mako-ai/mako](https://github.com/mako-ai/mako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

## objectstack

> <!-- GENERATED — DO NOT EDIT BY HAND. -->

<!-- GENERATED — DO NOT EDIT BY HAND. -->
<!-- Regenerate: pnpm --filter @objectstack/spec gen:liveness-counts -->

# `agent` — liveness counts (generated)

This type's row of the liveness state table, computed by the gate that enforces
it (`scripts/liveness/check-liveness.mts --json`, `types.<type>.byStatus`). Its
Notes prose is the `agent` row of [the ledger README](../README.md), which
also states the counting method. One file per governed type, and no total is
committed anywhere: `check:liveness` sums the shards when it reads them.
**Never hand-patch a number here** — fix the ledger or the schema and regenerate.

| Type | live | exp | elsewhere | dead | planned | classified |
|---|---|---|---|---|---|---|
| `agent` | 23 | 0 | 0 | 3 | 0 | 26 |

---
> Source: [objectstack-ai/objectstack](https://github.com/objectstack-ai/objectstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->

## fluid-mongo

> - Version tags have no `v` prefix (e.g. `1.9.0`, not `v1.9.0`).

# fluid-mongo

## Git

- Version tags have no `v` prefix (e.g. `1.9.0`, not `v1.9.0`).

## Architecture

- Single-module JVM library wrapping MongoDB Reactive Streams Java Driver into Kotlin coroutines/flows.
- Public API: interfaces in `sources-jvm/` (MongoClient, MongoDatabase, MongoCollection, ClientSession, flow types).
- Internal implementations: `ReactiveCoroutine*` classes delegate to reactive driver, converting Publishers to suspend/Flow.
- `MongoCollection.kt` (~1,500 lines) intentionally exceeds the 750-line guideline — it is a single interface mirroring the MongoDB driver API and cannot be split without fragmenting the public API surface.

---
> Source: [fluidsonic/fluid-mongo](https://github.com/fluidsonic/fluid-mongo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

## everyarrow

> Scope: repository root for the consolidated EveryArrow public platform.

# AGENTS.md

Scope: repository root for the consolidated EveryArrow public platform.

## Module boundaries

- `event-manager` owns transport adapters, registry orchestration, hooks, and the unified ServiceCall execution model.
- `dittoe-schema` owns the runtime-independent DITTOE grammar and Pipeline Metadata Document.
- `dittoe-engine` owns DITTOE pipeline execution and connector behavior.
- `utilities` owns reusable, product-independent infrastructure primitives.
- Keep operator UX and internal deployment systems outside this public repository unless explicitly approved.

Read the nearest module `AGENTS.md` before changing module code. Preserve the existing artifact coordinates and module-specific licenses.

## Build

Use Java 21. The root Maven reactor orders shared modules before their consumers. Add focused tests when behavior changes, then validate the affected module and the root reactor in proportion to risk.

---
> Source: [EveryArrow/everyarrow](https://github.com/EveryArrow/everyarrow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->

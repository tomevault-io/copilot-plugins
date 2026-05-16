## zod4-mock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Library:**

```bash
pnpm typecheck       # type-check
pnpm lint            # lint code
pnpm test            # run all tests
```

**Playground:**

```bash
pnpm check           # svelte-check
pnpm test:unit       # playground unit tests
pnpm test:component  # playground component tests
```

## Rules of Engagement

- **Scratch Files**: **NEVER** create scratch files.
- **Technical**: **NEVER USE `any`**.
- **Browsing**: Always use codesearch tools (e.g., `grep_search`) for browsing data.
- **Efficiency**: Only check relevant files for the given task.
- **Preparation**: Before starting any task, check the README and Wiki.
- **Testing**: Always write a test or component test when necessary.
- **Bugs**: If fixing a bug, always create a regression test.

## Architecture

This is a library (`zod4-mock`) that generates deterministic, schema-driven mock data from Zod v4 schemas. The public API is exported from [src/index.ts](src/index.ts).

### Core concepts

**World** — the central context for one generation session ([src/world.ts](src/world.ts)). Built fluently with `.withSubject()` and `.withSchema()`, then data is produced via `.generate()`. One world = one seed = one deterministic dataset.

**SubjectType** — an identity anchor representing a domain entity (Person, Company, TextFile, etc.). Defined with `defineSubjectType(name, zodObjectSchema)` ([src/subject.ts](src/subject.ts)). Subject instances get stable IDs (`person#1`, `person#2`, …) and their data is stored in the registry so matchers in other schemas can reference it.

**Generation pipeline** — for each field of a registered schema, values are resolved in this order:

1. **Matchers** (explicit functions provided in `world.withSchema(..., matchers)`)
2. **Key-based generators** — field name heuristics (e.g. `email`, `firstName`, fields ending in `id`) ([src/generators/key-based.ts](src/generators/key-based.ts))
3. **Schema-based generators** — Zod type introspection (string, number, enum, object, array, etc.) ([src/generators/schema-based.ts](src/generators/schema-based.ts))
4. `overrides` — deep merge after generation
5. `transform` — final transform function

**PRNG** — Mulberry32 seeded PRNG with FNV-1a hashing for per-field `fork(key)` derivation ([src/prng.ts](src/prng.ts)). Per-field seeding means adding/removing schema fields does not disturb values for other fields. The `Prng.fork(key)` method creates an independent child PRNG without consuming the parent's state.

**Registry** — in-memory store for all generated data within a world ([src/registry.ts](src/registry.ts)). Matchers can call `ctx.registry.pick<T>('typename')` to reference data generated for other subject types, enabling cross-API consistency.

### Zod v4 internals

Zod v4 stores schema definitions at `schema._zod.def` (not `schema._def` as in v3). Checks are class instances accessed via `check._zod.def`. The codebase accesses these directly via type-casting — this is intentional since Zod v4 doesn't expose a stable public introspection API.

### Test structure

- `tests/unit/` — isolated unit tests for each module
- `tests/integration/` — three full-scenario integration tests (document-corpus, invoicing, media-library), each with its own `schemas.ts` and `world.ts`. The media-library test demonstrates multi-subject, cross-API consistency.

### Documentation rule

**Whenever a public API changes — new method, removed method, changed signature, or changed behaviour — update `wiki/api-reference.md` in the same step.** Do not defer documentation to a follow-up; it will be forgotten.

### TypeScript strictness

The `tsconfig.json` enables `exactOptionalPropertyTypes` and `noUncheckedIndexedAccess`. Array indexing always returns `T | undefined`, requiring `!` assertions or null checks. All imports use `.js` extensions (Node16 ESM module resolution).

---
> Source: [dxlbnl/zod4-mock](https://github.com/dxlbnl/zod4-mock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->

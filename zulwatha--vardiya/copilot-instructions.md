## vardiya

> vardiya is a SQLite-backed job queue for Node.js. One runtime dependency (`better-sqlite3`). This repo is split so several agents can work in parallel without stepping on each other.

# Agent ownership map

vardiya is a SQLite-backed job queue for Node.js. One runtime dependency (`better-sqlite3`). This repo is split so several agents can work in parallel without stepping on each other.

## Module owners

| Area | Path | Owner | Responsibility |
|------|------|-------|----------------|
| Types / contract | `src/types.ts` | Lead (frozen) | Shared interfaces and types. Do not edit. |
| Public API stubs | `src/index.ts`, `src/vardiya.ts` | Lead (signatures frozen) | Export surface. Fill bodies only; do not change method signatures. |
| Utilities | `src/util/` | Shared | Small pure helpers (ids, sleep, typed emitter). |
| Storage | `src/storage/` | Agent A | SQLite schema, migrations, `Storage` implementation. |
| Worker | `src/worker/` | Agent B | Claim loop, handlers, concurrency, heartbeats, events. |
| Scheduler | `src/scheduler/` | Agent C | In-house cron parsing, repeatables, delayed promotion ticks. |
| Tests & benches | `test/`, `bench/` | Agent D | Vitest tests and benchmarks. |

Stay inside your directory unless you need a tiny shared helper in `src/util/`. Prefer asking for a util over inventing a second copy.

## Frozen types rule

`src/types.ts` is frozen after the foundation task. If you believe a change is required, do **not** edit the definitions. Add a comment block like this near the call site or in your PR notes:

```ts
/*
 * PROPOSED-CHANGE:
 * What: add Storage.reclaimStale(now, staleAfterMs)
 * Why: workers die mid-job and active rows need reclaim
 * Suggested shape: reclaimStale(now: number, staleAfterMs: number): number
 */
```

The lead merges contract changes deliberately so parallel work does not break.

## Stack reminders

- Node >= 22, TypeScript strict, dual ESM+CJS via tsup.
- Runtime dependency: `better-sqlite3` only. Cron, backoff, and ids are in-house.
- Lint/format with biome. Tests with vitest.
- `npm run build`, `npm run lint`, and `npm test` should stay green.

## Writing style

Binding for every agent and every file: code comments, TSDoc, READMEs, docs, commit messages, and error messages.

Never use the em dash character (—) or en dash (–) anywhere. Use a comma, a period, or parentheses instead.

Write like a pragmatic engineer explaining things to a colleague, not like an AI. Short sentences. Contractions are fine. First person is fine.

Banned filler vocabulary: "delve", "seamlessly", "robust", "leverage", "cutting-edge", "empower", "streamline", "comprehensive", "In today's world", "It's important to note", "Additionally," as a sentence opener.

No hype, no exclamation marks in docs. State what the thing does and why, plainly. Prefer one concrete example over three adjectives.

Don't turn everything into bullet lists. Prose is allowed and often better.

---
> Source: [Zulwatha/vardiya](https://github.com/Zulwatha/vardiya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

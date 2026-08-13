## kassette

> Embeddable durable execution for agentic workflows: at-most-once journaling, suspend/resume, parallel branches, and forking — without a runtime, a server, or external infrastructure. The journal is the single source of truth and survives any process death; replay rebuilds full state from the journal alone, so re-invocation after a crash skips completed work and resumes live at the first unfinished step. See [README.md](README.md) for the full overview.

# kassette

Embeddable durable execution for agentic workflows: at-most-once journaling, suspend/resume, parallel branches, and forking — without a runtime, a server, or external infrastructure. The journal is the single source of truth and survives any process death; replay rebuilds full state from the journal alone, so re-invocation after a crash skips completed work and resumes live at the first unfinished step. See [README.md](README.md) for the full overview.

## Design Principles

Ranked. When two principles conflict, the higher one wins.

1. **Crash Safety** — The journal must survive any process death. Replay must recover full state from the journal alone. Re-invocation is the caller's job — but the primitives must make it trivial.
2. **Journal Authority** — Completed work is settled. The journal's record of a step is authoritative — don't re-execute, don't invalidate, don't expire.
3. **Single Source of Truth** — All state lives in the journal. Everything else is cache.
4. **Observability** — The journal is the only reliable record of what happened. It must be readable without special tooling, and complete enough to explain a run to a reader who wasn't there.
5. **Embed Without Distortion** — The host code and framework shouldn't need to know the durability layer exists. Don't force deterministic code, don't break streaming, don't require restructuring.
6. **Minimal Surface Area** — Resist scope creep. If the caller can achieve it with the existing API (`step()`, `suspend()`, `parallel()`), don't add API for it.

## Packages

- `packages/core` (`@usekassette/core`) — storage backend, append-only journal, record/replay primitive.
- `packages/kassette` (`@usekassette/kassette`) — workflow API: `kassette()`, `Context`, `step`, `suspend`, `parallel`, `fork`.
- `packages/s3` (`@usekassette/s3`) — S3 storage backend.
- `packages/cli` (`@usekassette/cli`) — CLI for inspecting and forking journals.

## Examples

See `examples/` — each subdirectory has its own README.

## External References

| Need                                | File                            |
| ----------------------------------- | ------------------------------- |
| Overview, properties, guarantees    | `README.md`                     |
| Architecture, replay, fencing       | `docs/architecture.md`          |
| API, journal format, errors         | `docs/reference.md`             |
| Dispatcher/retry wiring             | `docs/wiring-dispatcher.md`     |
| Storage backend choice              | `docs/storage-backends.md`      |
| Object storage CAS design           | `docs/object-storage-design.md` |
| Concurrency and step naming         | `docs/concurrency.md`           |
| Replay-compatible versioning        | `docs/versioning.md`            |
| Operations, CLI, forking, retention | `docs/operations.md`            |

## Commands

Use `pnpm` (not npm/npx) to run all commands in this project.

- `pnpm check` — lint, format, type check.
- `pnpm test` — run all unit tests (vitest).
- `pnpm test --project <name>` — run tests for one package.
- `pnpm test -- path/to/file.test.ts` — run a single test file.

## Key Conventions

- Tests live alongside source code as `*.test.ts`.
- Build outputs under `packages/*/dist/` are generated; edit `src/` and rebuild with `pnpm -r --filter './packages/*' build`.

## Agent skill

When operating on a kassette journal — inspecting, debugging, forking, or resuming a run — read [`skills/kassette/SKILL.md`](skills/kassette/SKILL.md) first. It covers storage discovery, journal concepts, `jq` patterns, status interpretation, and safe forking with the `kassette` CLI.

## Verification

When changing core session/storage code, the streaming pattern, journal entry format, core types, the `Storage` interface, or public API: run `pnpm check && pnpm test`, and re-read the **Properties** and **Guarantees** sections of [README.md](README.md) against your diff — those contracts must continue to hold.

---
> Source: [lostinpatterns/kassette](https://github.com/lostinpatterns/kassette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

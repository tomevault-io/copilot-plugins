## primer

> Primer has completed Phases 1 through 7, the external-connector readiness gate, and a scoped Level 2 query-planning enhancement. Development is paused for manual live testing under `docs/manual-live-testing.md`; do not begin Phase 8 or add live connectors without a new explicit direction. The complete CLI pipeline, local HTTP API, account/content operations, streamed grounded chat, evidence navigation, account-scoped retrieval inspection, evaluation reporting, `primer.connector.v1`, atomic synchronization, diagnostics, backup, readiness gates, and bounded answer-query planning are implemented. The product continues to use only local fixture data. The `acme-v0.1` full and later targeted live runs are preserved; `acme-v0.3` is the active verified fixture. Browser code must not access SQLite or provider credentials, and chat must derive its actor only from the active server session. Primer does not index source-code bodies; Helix/Pi owns real repository exploration.

# Primer agent guide

Primer has completed Phases 1 through 7, the external-connector readiness gate, and a scoped Level 2 query-planning enhancement. Development is paused for manual live testing under `docs/manual-live-testing.md`; do not begin Phase 8 or add live connectors without a new explicit direction. The complete CLI pipeline, local HTTP API, account/content operations, streamed grounded chat, evidence navigation, account-scoped retrieval inspection, evaluation reporting, `primer.connector.v1`, atomic synchronization, diagnostics, backup, readiness gates, and bounded answer-query planning are implemented. The product continues to use only local fixture data. The `acme-v0.1` full and later targeted live runs are preserved; `acme-v0.3` is the active verified fixture. Browser code must not access SQLite or provider credentials, and chat must derive its actor only from the active server session. Primer does not index source-code bodies; Helix/Pi owns real repository exploration.

This file is an entrypoint, not the full specification.

## Related projects

| Project | Local path | Responsibility |
|---|---|---|
| Primer | `~/Desktop/acme/primer` | Knowledge product and fictional Acme evidence corpus; outside the Issues → Helix runtime loop. |
| Acme Identity | `~/Desktop/acme/acme-identity` | Optional suite principal provider; Primer still owns actor mappings and knowledge ACLs. |
| Prelude | `~/Desktop/acme/prelude` | Project inception drafting; may query Primer over HTTP and export Helix bootstrap artifacts. |
| Helix | `~/Desktop/acme/helix` | Agent workflow control plane that receives work and orchestrates changes. |
| Acme Issues | `~/Desktop/acme/acme-issues` | Local issue tracker and webhook harness that triggers Helix and receives callbacks. |
| Acme Projects | `~/Desktop/acme/acme-projects` | Feature-idea and collaboration board for existing Helix repos; can manually create non-triggering issues through Acme Issues. |
| Acme Todo | `~/Desktop/acme/acme-todo` | Disposable target application used for agent implementation and verification. |

Existing-repo runtime flow: Acme Issues → Helix → Acme Todo, followed by a Helix completion callback to Acme Issues. Primer remains outside that path while its CLI and web product are built. Later, Acme Issues may be a read-only authoritative source for Primer, and Helix may consume bounded Primer evidence through a stable query boundary.

Intended feature flow for existing repos begins in Acme Projects, which requests a linked implementation issue from Acme Issues; Acme Issues alone triggers Helix. New-project inception belongs to Prelude, which may query Primer over HTTP and exports bootstrap artifacts for Helix empty-workspace bootstrap.

## Read first

1. [`README.md`](./README.md) for project status and document routing.
2. [`modern-knowledge-base.md`](./modern-knowledge-base.md) for the original concept.
3. [`docs/vision.md`](./docs/vision.md) and [`docs/product-spec.md`](./docs/product-spec.md) before changing product scope.
4. [`docs/architecture.md`](./docs/architecture.md) before choosing infrastructure, models, connectors, data stores, or service boundaries.
5. [`docs/evaluation.md`](./docs/evaluation.md) before claiming a retrieval or answer-quality improvement.
6. [`docs/decisions.md`](./docs/decisions.md) before turning an assumption into a durable choice.

## Invariants

- Primer is a focused knowledge product whose first release proves a complete, trustworthy evidence pipeline before expanding its operational breadth.
- Original systems are authoritative; the search index is derived and rebuildable.
- Different source types retain source-aware processing and provenance.
- Authorization filtering happens before evidence reaches a model.
- Suite permissions and knowledge access are separate: Identity gates Primer operations, while a mapped Primer actor and Primer-owned groups gate evidence.
- Query planning may vary search text only. It cannot choose the actor, groups, project scope, ACL filters, evidence, or answer; every planned query runs against the same pre-authorized population.
- Retrieval remains inspectable from candidates through final evidence.
- Generated claims must link to supporting evidence or state uncertainty.
- Exact search and semantic search are complementary; neither is treated as truth.
- Domain rules, source authority, freshness, and supersession remain explicit.
- MCP and live third-party integrations are optional boundaries, not MVP foundations.
- The CLI is the first product surface and owns no duplicate business logic; the later API and web UI reuse the same application services.
- OpenRouter is the initial provider for chat and embeddings. Use the official OpenRouter TypeScript SDK for embeddings and Vercel AI SDK for grounded chat; streaming remains deferred. Pi is reserved for a later server-side, read-only UI simulation; Helix owns Pi in real implementation workflows.
- Primer supplies bounded organizational context and non-authoritative code leads. It does not duplicate Helix's current-code exploration or automatically index Pi output.
- The current implementation remains index-first. Future source-native discovery is an extension path, not current scope, and may never bypass authorization, normalization, provenance, or evidence construction.
- Acme Issues remains authoritative for issues, comments, and run lineage. Primer may index it later but does not mutate it.
- Helix remains responsible for workflow orchestration. Primer may later supply evidence but does not absorb Helix's agent loop.
- Product direction, planned work, and implemented behavior must be labeled separately.
- Keep `PRIMER_AUTH_PROVIDER=standalone` as the default. External auth belongs only in the HTTP host through Primer's replaceable plain-HTTP adapter; the CLI remains independently usable with explicit fixture actors.

## Change discipline

- Preserve `modern-knowledge-base.md` as the original concept unless explicitly asked to revise it.
- Record consequential choices and reversals in `docs/decisions.md`.
- Keep unresolved implementation details technology-neutral; do not reopen settled decisions without recording the reversal and consequence.
- Do not describe planned behavior as shipped behavior.
- Favor a small, inspectable vertical slice over breadth of connectors or infrastructure.
- Keep CLI text output useful for people and add stable `--json` output where the contract will later serve the API or another consumer.
- Do not let provider SDK types spread through source, retrieval, authorization, or evaluation modules; contain them at the model adapter boundary.

## Current implementation map

```text
src/cli.ts          CLI adapter, stable JSON surfaces, and the serve entrypoint
src/app.ts          Express app: HTTP API, HttpOnly sessions, and web delivery
src/webAssets.ts    Vite middleware in dev, built dist/web in production
src/services.ts     application use cases, retrieval, fusion, evidence, evaluation
src/database.ts     SQLite schema, record writer, FTS, traces, evaluation runs
src/connectors/     independent acquisition contracts, registry, local connectors,
                    semantic HTTP providers, and canonical artifact processing
src/markdown.ts     frontmatter and heading-aware Markdown processor
src/slack.ts        deterministic Slack thread processor
src/fixture.ts      Acme fixture and stable-identity validator
src/embeddings.ts   deterministic test adapter and official OpenRouter SDK adapter
src/planner.ts      bounded deterministic and Vercel/OpenRouter query-planner adapters
src/types.ts        domain and retrieval contracts
web/                React/Vite account and content operations consuming only the API
                    plus chat, evidence, retrieval trace, and evaluation surfaces
test/               fixture, connector, lifecycle, HTTP, retrieval, authorization, evaluation, CLI tests
```

Use `node --import tsx` rather than the `tsx` executable when the environment forbids tsx's IPC listener. Normal verification is `npm run verify` (`typecheck`, `test`, `build`). A fresh clone needs the nested fixture repositories; `npm test` restores them first through `npm run fixtures:restore`.

## Shared Acme stack

Primer runs the same foundation as Helix, Prelude, Acme Projects, and Acme Issues: Node.js with TypeScript and ESM, Express 5 hosting a React 19/Vite 8 UI, `better-sqlite3` with WAL and foreign keys, and `node:test` through `tsx`. Each project exposes `<name> serve` from its CLI, serves the UI from source under `<NAME>_DEV=1` through Vite middleware, and builds to `dist/web`. Keep `src/app.ts`, `src/webAssets.ts`, `typecheck`, `verify`, and the `83xx` port assignment (Primer is 8317) consistent with those projects; raise stack changes across the set rather than diverging here.

Primer, Prelude, Acme Projects, and Acme Issues require Node.js 20.19+. Helix requires 22.19+ because the Pi SDK does, so the set has no single floor yet; do not assume Node 22 features here until that is settled across the projects.

---
> Source: [eimg/primer](https://github.com/eimg/primer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->

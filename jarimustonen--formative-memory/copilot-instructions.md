## formative-memory

> OpenClaw plugin implementing a biologically-inspired associative memory system. Standalone repo; installs as an OpenClaw extension.

# OpenClaw Associative Memory Plugin

OpenClaw plugin implementing a biologically-inspired associative memory system. Standalone repo; installs as an OpenClaw extension.

## Issue tracking

Issues live in `issues/` and are managed with [`issuectl`](https://github.com/jarimustonen/issuectl). Use the `/issue` skill or `issuectl list` from the CLI. See `.issuectl/AGENTS.md` for policy.

## Gitignored directories

- `history/` — agent scratchpad and ephemeral planning docs (not tracked)

## Versioning / openclaw SDK compat

`package.json` has four distinct openclaw version fields. They are NOT
interchangeable — bumping them together when you only need one causes
needless breaking changes for downstream hosts:

- `devDependencies.openclaw` — version installed for local typecheck/build.
- `openclaw.build.openclawVersion` — built-with marker, mostly metadata.
- `peerDependencies.openclaw` — minimum host package version consumers may
  install with.
- `openclaw.compat.pluginApi` — minimum **runtime** host API the plugin
  requires; the host rejects loading if its API is older.

Type-only imports (`import type { ... } from "openclaw/plugin-sdk/types"`)
are erased at compile time and produce no runtime reference. They only
require bumping the **dev** fields, never `peerDependencies` or
`compat.pluginApi`. Verify with `grep -l "openclaw/plugin-sdk/<subpath>" dist/*.js`
after `pnpm build` — no match means the import is dev-only.

### Two parallel embedding-provider contracts — don't confuse them

The host exposes **two separate** embedding-provider APIs. This plugin
consumes the **memory** one and must not be migrated to the other:

- **Memory contract (what we use):**
  `openclaw/plugin-sdk/memory-core-host-engine-embeddings` →
  `MemoryEmbeddingProvider` (`embedQuery` + `embedBatch`),
  `getMemoryEmbeddingProvider`, `listMemoryEmbeddingProviders`,
  `MemoryEmbeddingProviderAdapter`.
- **Generic contract (added in openclaw#84947, v2026.5.22 — NOT ours):**
  `openclaw/plugin-sdk/embedding-providers` → `EmbeddingProvider`
  (`embed` + `embedBatch`), registered via the `embeddingProviders`
  plugin capability.

`#84947` is **additive**: it added the generic subpath alongside the
memory one; it did not change or deprecate the memory path, which is
unchanged through v2026.5.26. An issue titled "adapt to the embedding
provider contract" does **not** mean migrate to `embedding-providers` —
verify against the actual import the plugin uses before assuming a
breaking change.

## Documentation

- `docs/architecture.md` — system architecture overview
- `docs/how-memory-works.md` — how the memory system works
- `docs/glossary.md` — terminology reference
- `docs/openclaw-memory-coexistence.md` — comparison and coexistence with OpenClaw built-in memory
- `docs/v0.6/compat-audit.md` — v0.6 milestone gate: static-analysis compat audit for OpenClaw v2026.5.26 (live boot still pending)
- `docs/v0.6/codex-applicability.md` — v0.6 decision: Codex harness path is not applicable to Sylvia/Metka (both run deepseek, no codex harness plugin)
- `docs/v0.6/compaction-measurements.md` — consolidation timing measurements + the chunk-vs-timeout decision for `@compaction-safety`
- `docs/integration-test-strategy.md` — integration test harness strategy + pin-bump checklist

## Conventions

- **Workspace `"."` placeholder** — `getWorkspace(".")` in `src/index.ts` means "no preference" and aliases to whatever directory was already bound by an earlier caller. Commands and ctx-less factory invocations rely on this. If you add a new caller that needs a specific workspace path, pass it through — don't hardcode a different default string, or the fail-fast mismatch check in `getWorkspace` will reject legitimate caller combinations.

- **Recall must never produce silent false negatives** — any embedding
  failure in `MemoryManager.search()` is caught and falls back to BM25
  (never rethrown); `lastDegradedAt` is set on the manager so callers
  can check `manager.wasRecentlyDegraded(windowMs)` and surface a
  notice. The signal lives on the manager, NOT the circuit breaker —
  the breaker's `consecutiveFailures` is wiped by intervening probe
  successes, but the user-facing degraded-mode notice must latch
  across them. The notice itself is intentionally NOT cached in
  `assemble()`'s entry — `applyBm25Notice()` re-applies it freshly on
  every return so a transient failure surfaces in the same turn it
  occurred and recovery clears it without prompt-cache thrash. The
  same principle still needs to land on `store()` / autoCapture — see
  the `store-quota-silent-drop` issue.

---
> Source: [jarimustonen/formative-memory](https://github.com/jarimustonen/formative-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->

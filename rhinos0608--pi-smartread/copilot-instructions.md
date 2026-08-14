## agents-md

> Pi-SmartRead is the Pi coding agent's code-intelligence extension — unified file reading, structural file analysis (callers, inheritance, overrides, quality signals), hybrid code search (BM25 + AST symbol + semantic), repo mapping, and call graph analysis. It relies on one shared sister codebase for serialized workspace-evidence contracts and serves as the RPC resolver for Pi-SmartEdit's evidence authorization.

# Agent Reference: Pi-SmartRead Sister Repos

Pi-SmartRead is the Pi coding agent's code-intelligence extension — unified file reading, structural file analysis (callers, inheritance, overrides, quality signals), hybrid code search (BM25 + AST symbol + semantic), repo mapping, and call graph analysis. It relies on one shared sister codebase for serialized workspace-evidence contracts and serves as the RPC resolver for Pi-SmartEdit's evidence authorization.

## `/Users/rhinesharar/Pi-Workspace-Protocol`

- **Package:** `@rhinos0608/pi-workspace-protocol` (pinned in `package.json` as `github:rhinos0608/Pi-Workspace-Protocol#v0.3.0`).
- **Purpose:** Versioned TypeScript contracts, SHA-256/id helpers, runtime validators, and an event-bus RPC layer for the SmartRead/SmartEdit inspect+patch protocol.
- **What Pi-SmartRead consumes:**
  - `src/inspect.ts` imports `PROTOCOL_SCHEMA_VERSION`, `hashSessionFilePath`, `inspectionIdFor`, `resourceIdFor`, `canonicalizeWorkspaceRoot`, `WorkspaceEvidenceEnvelope`, `InspectedResource`, `InspectMode` to build schema-3 evidence envelopes.
  - `src/workspace-evidence-resolver.ts` imports `PROTOCOL_SCHEMA_VERSION`, `validateInspectionEnvelope`, `hashSessionFilePath`, `WorkspaceEvidenceEnvelope` to validate and cache published envelopes.
  - `src/read-many.ts` imports `canonicalizeWorkspaceRoot`, `hashSessionFilePath`, `inspectionIdFor`, `PROTOCOL_SCHEMA_VERSION`, `InspectedResource`, `WorkspaceEvidenceEnvelope` to build batch workspace-evidence envelopes from multi-file reads.
  - `src/mcp-registry.ts` imports `RPC_CHANNELS` to create the evidence resolver on the `inspectPatch` channel.
  - `src/path-evidence.ts` imports `sha256OfString` for per-file content hashing in path-mode evidence.
  - `src/index.ts` receives evidence from the wrapped `read`, `inspect`, and `grep` tools, publishing envelopes into the shared `EvidenceResolver`.

## `/Users/rhinesharar/Pi-SmartEdit`

- **Package:** Pi-SmartEdit (the `edit`/`write` extension).
- **Purpose:** Provides the `edit` and `write` tools; validates edits against workspace-evidence envelopes produced by Pi-SmartRead.
- **What Pi-SmartRead provides for Pi-SmartEdit:**
  - **RPC evidence resolver** (`src/workspace-evidence-resolver.ts`): Answers `resolve_evidence` RPC requests on `RPC_CHANNELS.inspectPatch`. Pi-SmartEdit's `patch.ts` sends a request with `inspectionId`/`sessionFilePath`/`workspaceRoot`, and the resolver returns the cached `WorkspaceEvidenceEnvelope`. The resolver cache is rebuilt from `tool_result` events (`pi.tool_result.inspect`, `pi.tool_result.read`, and `pi.tool_result.grep`) seen on the event bus — tool result details are the durable source of truth.
  - **Inspect tool** (`src/inspect-tool.ts`, `src/inspect.ts`): Two modes — directory (ranked repo map plus clusters, layers, boundaries, routes, hotspots, graphSchema, deadCode) and file (structural facts + quality signals plus callDepth/callDirection, impact, deadCode, diff, hotspots, routes, graphSchema). Directory mode returns envelope mode `map` with zero resources (no file authorization). File mode returns envelope mode `symbol` with per-referenced-symbol `coverage: "search-match"` (weak evidence; must read before editing). Query/symbol/action modes removed; use `grep` tool for code search.
  - **Grep tool** (`src/grep-tool.ts`): Primary code search — BM25 ranking + AST symbol matching + embedding semantic fallback behind a grep-shaped interface, with optional graph-aware filtering (`graphFilter` param). Returns envelope mode `query` with `coverage: "search-match"` per hit. `tool_result.grep` events feed the resolver cache.
  - **Read tool wrapper** (`src/index.ts:432-436`, `src/hook.ts`): Owns single-file provenance plus `{ paths: [...] }`, `{ query }`, and `{ symbol }` dispatch. `symbol` param resolves qualified symbol names via LSP or the context graph. Query mode uses the shared semantic index, then reads selected files through the same evidence-emitting single-read path.
  - **Read-many batch** (`src/read-many.ts`): Internal packing engine for multi-file/query-selected reads. `read_files` is no longer registered. Batch evidence includes only complete file blocks actually rendered; partial and omitted blocks are not authorized.
  - **Semantic index** (`src/semantic-index.ts`, `src/semantic-index-registry.ts`): Startup-warmed, ignore-aware, model-fingerprinted SQLite vector cache with independent incremental file state. Query ranking fuses whole-corpus BM25 and embedding ranks with RRF; grep+AST is availability/error fallback only.
  - **Context hygiene** (`src/context-hygiene.ts`): Tracks read results, marks stale context after mutations. Pi-SmartEdit's edit results trigger hygiene events via the event bus.
  - **Context graph** (`src/context-graph.ts`, `src/graph-mutate.ts`): `graph_mutate` receives `breakage`/`co-change` edges (from Pi-SmartEdit's post-edit evidence pipeline or manual tool calls) and persists them via the `EdgeStore` for future graph-aware retrieval.

## `/Users/rhinesharar/Pi-SmartEdit` (what it consumes from Pi-SmartRead)

Pi-SmartEdit's `AGENTS.md` states it consumes:
- `src/index.ts` subscribes to `tool_result` for `read`, `read_files`, `read_multiple_files`, and `intent_read`, then calls `recordRead`/`recordReadSession` from `src/core/read-cache.ts` so edits are allowed only against seen/read files.
  - **In Pi-SmartRead**, the equivalent is `src/file-read-cache.ts` (`recordContiguous`, `recordSparse`, `getSnapshot`, `invalidate`, `clearSession`, `resolveSessionKey`), which stores per-session file snapshots for anchor-stale recovery. Pi-SmartEdit's `recordRead`/`recordReadSession` references should map to this module.
- `src/patch.ts` queries Pi-SmartRead's resolver over `RPC_CHANNELS.inspectPatch` for evidence authorization.
  - **In Pi-SmartRead**, this is served by `src/workspace-evidence-resolver.ts` and wired at `src/index.ts:441-450` via `installInspectAndResolver(bus)`.

## Coordination note

`@rhinos0608/pi-workspace-protocol` has no independent versioning gate beyond the git tag pin in `package.json`. Changes to shared types (`WorkspaceEvidenceEnvelope`, `InspectedResource`, `EvidenceRef`, etc.) must be version-bumped and accompanied by compatible consumer updates in both Pi-SmartEdit and Pi-SmartRead.

The evidence flow is: Pi-SmartRead produces envelopes → stores them in an in-memory resolver cache → Pi-SmartEdit requests them via RPC → validates coverage and freshness before authorizing edits. The event bus is the transport; tool result `details.workspaceEvidence` is the durable source of truth — the resolver cache is derived, never parsed from rendered text.

## Operational Contracts and Invariants

### ❌ Cross-root file reads are INTENTIONALLY unrestricted — do NOT wire boundary enforcement into read paths
This is the most important operational rule in this codebase.

`src/workspace-boundary.ts` exports `isWithinRoot`, `getAllowedRoot`, and `resolveWorkspacePath` — but these functions are **only used for semantic-index/retrieval scoping**, NOT for gating file read, inspect, or grep operations. Git history (commits `9426efc`, `34208d1`) shows cross-root read permissions were deliberately made external to this codebase, and tests lock in this behavior.

Do NOT wire `isWithinRoot`/`getAllowedRoot`/`resolveWorkspacePath` into:
- `src/read-many.ts` (`resolveExplicitFile`)
- `src/hook.ts` (contextual read intercept)
- `src/inspect.ts` (file inspect)

`PI_SMARTREAD_ALLOWED_ROOT` scopes the semantic index, background indexing, and retrieval — it does NOT gate direct reads. If someone proposes 'fixing' this as a security issue, it is already a known, tested, intentional boundary. Escalate rather than silently reversing it.

### canonicalPath MUST be a true realpath (symlinks resolved) — evidence contract
`src/inspect.ts`, `src/grep-tool.ts`, and `src/hook.ts` resolve file paths to `canonicalPath` for evidence envelopes. These envelopes are consumed by Pi-SmartEdit (via Pi-Workspace-Protocol's contract) for SHA-256 freshness verification. The `canonicalPath` MUST be a `realpathSync` result (symlinks resolved).

The canonical pattern is `tryCanonical(path)` which calls `realpathSync` with a try/catch fallback for non-existent files. All evidence-producing code paths now follow this pattern. Do not revert to bare `path.resolve`/`path.join` — this will break the shared evidence contract with Pi-SmartEdit.

### cosineSimilarity — single source of truth
`src/scoring.ts:151-163` is the single canonical implementation. It returns `0` for zero-length vectors and length mismatches (not `-Infinity`, which poisons downstream RRF ranking). `src/rerank.ts` and `src/search-tool.ts` import it — they do not have their own copies. Do not add a new local implementation.

### Directory-mode dead code detection uses relative paths
`src/inspect.ts:203` calls `detectDeadCode(pathRelative(cwd, pathResolve(cwd, input.path)), callGraph)` — the relative path is required because `callGraph.functions[].file` stores relative paths. File mode at line 649 uses the same pattern. Do not pass absolute paths to `detectDeadCode`.

### Workspace-boundary tests verify non-enforcement
`test/unit/workspace-boundary.test.ts` has tests confirming `resolveWorkspacePath` allows outside-path resolution without the `PI_SMARTREAD_ALLOWED_ROOT` env var. These tests explicitly DO NOT assert path restriction — they assert permission-external behavior. If boundary enforcement is ever added to read paths, these tests must be updated to reflect the new behavior, not just deleted.

---
> Source: [rhinos0608/Pi-SmartRead](https://github.com/rhinos0608/Pi-SmartRead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

## agents-md

> Pi-SmartRead is the Pi coding agent's code-intelligence extension — unified multi-mode file inspection, intent-based retrieval, repo mapping, symbol search, cross-file resolution, and call graph analysis. It relies on one shared sister codebase for serialized workspace-evidence contracts and serves as the RPC resolver for Pi-SmartEdit's evidence authorization.

# Agent Reference: Pi-SmartRead Sister Repos

Pi-SmartRead is the Pi coding agent's code-intelligence extension — unified multi-mode file inspection, intent-based retrieval, repo mapping, symbol search, cross-file resolution, and call graph analysis. It relies on one shared sister codebase for serialized workspace-evidence contracts and serves as the RPC resolver for Pi-SmartEdit's evidence authorization.

## `/Users/rhinesharar/Pi-Workspace-Protocol`

- **Package:** `@rhinos0608/pi-workspace-protocol` (pinned in `package.json` as `github:rhinos0608/Pi-Workspace-Protocol#v0.3.0`).
- **Purpose:** Versioned TypeScript contracts, SHA-256/id helpers, runtime validators, and an event-bus RPC layer for the SmartRead/SmartEdit inspect+patch protocol.
- **What Pi-SmartRead consumes:**
  - `src/inspect.ts` imports `PROTOCOL_SCHEMA_VERSION`, `hashSessionFilePath`, `inspectionIdFor`, `resourceIdFor`, `canonicalizeWorkspaceRoot`, `WorkspaceEvidenceEnvelope`, `InspectedResource`, `InspectMode` to build schema-3 evidence envelopes.
  - `src/workspace-evidence-resolver.ts` imports `PROTOCOL_SCHEMA_VERSION`, `validateInspectionEnvelope`, `hashSessionFilePath`, `WorkspaceEvidenceEnvelope` to validate and cache published envelopes.
  - `src/read-many.ts` imports `canonicalizeWorkspaceRoot`, `hashSessionFilePath`, `inspectionIdFor`, `PROTOCOL_SCHEMA_VERSION`, `InspectedResource`, `WorkspaceEvidenceEnvelope` to build batch workspace-evidence envelopes from multi-file reads.
  - `src/mcp-registry.ts` imports `RPC_CHANNELS` to create the evidence resolver on the `inspectPatch` channel.
  - `src/path-evidence.ts` imports `sha256OfString` for per-file content hashing in path-mode evidence.
  - `src/index.ts` receives evidence from the wrapped `read` tool and `inspect` tool, publishing envelopes into the shared `EvidenceResolver`.

## `/Users/rhinesharar/Pi-SmartEdit`

- **Package:** Pi-SmartEdit (the `edit`/`write` extension).
- **Purpose:** Provides the `edit` and `write` tools; validates edits against workspace-evidence envelopes produced by Pi-SmartRead.
- **What Pi-SmartRead provides for Pi-SmartEdit:**
  - **RPC evidence resolver** (`src/workspace-evidence-resolver.ts`): Answers `resolve_evidence` RPC requests on `RPC_CHANNELS.inspectPatch`. Pi-SmartEdit's `patch.ts` sends a request with `inspectionId`/`sessionFilePath`/`workspaceRoot`, and the resolver returns the cached `WorkspaceEvidenceEnvelope`. The resolver cache is rebuilt from `tool_result` events (`pi.tool_result.inspect` and `pi.tool_result.read`) seen on the event bus — tool result details are the durable source of truth.
  - **Inspect tool** (`src/inspect-tool.ts`, `src/inspect.ts`): Multi-mode retrieval (path/query/symbol/map) that emits `details.workspaceEvidence` envelopes consumed by patch for authorization. Path mode produces **strong** (full-file/line-range) evidence; query/symbol/map modes produce **weak** (search-match) evidence.
  - **Read tool wrapper** (`src/index.ts:432-436`): Wraps the builtin `read` tool to emit workspace-evidence envelopes alongside every read result, making single-file reads patch-authorizable without requiring a separate `inspect` call.
  - **Read-many batch** (`src/read-many.ts`): `read_files` aggregates per-file evidence into a single batch envelope so multi-file reads are patch-authorizable.
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

---
> Source: [rhinos0608/Pi-SmartRead](https://github.com/rhinos0608/Pi-SmartRead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->

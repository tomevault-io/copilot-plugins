## agents-md

> `@rhinos0608/pi-workspace-protocol` provides versioned TypeScript contracts, SHA-256/id helpers, runtime validators, and an event-bus RPC layer shared by Pi-SmartRead (evidence producer) and Pi-SmartEdit (evidence consumer).

# Agent Reference: Pi Workspace Protocol

`@rhinos0608/pi-workspace-protocol` provides versioned TypeScript contracts, SHA-256/id helpers, runtime validators, and an event-bus RPC layer shared by Pi-SmartRead (evidence producer) and Pi-SmartEdit (evidence consumer).

## Sister Repos

### `/Users/rhinesharar/Pi-SmartRead` (producer)
- Builds `WorkspaceEvidenceEnvelope` instances via `src/inspect.ts`, `src/read-many.ts`, `src/grep-tool.ts`.
- Publishes envelopes into a resolver cache; serves `resolve_evidence` RPC on `RPC_CHANNELS.inspectPatch`.
- Envelope mode: directory inspect → `map` (zero resources, no file auth); file inspect → `symbol` (search-match, weak evidence); grep/read → per-hit coverage.

### `/Users/rhinesharar/Pi-SmartEdit` (consumer)
- Requests `resolve_evidence` via `RPC_CHANNELS.inspectPatch` in `src/patch.ts`.
- Validates `sessionId` (via `hashSessionFilePath`), `canonicalWorkspaceRoot`, resource coverage (`full-file`/`line-range`), and `fullFileSha256` freshness before authorizing edits.

## Schema Versioning

**`PROTOCOL_SCHEMA_VERSION = 3`** (`src/types.ts:6`). Enforced by exact match in `src/contract.ts` — no version negotiation or range checks. Any version bump is a breaking change; both consumers (Pi-SmartRead *and* Pi-SmartEdit) must update in lockstep. Package semver (`0.4.0`) tracks API surface, not wire format. `0.4.0` adds the additive `languageIntelligence` RPC channel (`pi.workspace.language_intelligence.rpc`) with `language_intelligence_capabilities` / `check_post_edit_diagnostics` — `PROTOCOL_SCHEMA_VERSION` stays `3`.

## Operational Contracts and Invariants

### canonicalPath MUST be a true realpath (symlinks resolved)
Evidence envelopes attest to `canonicalPath` as the `realpathSync` result. Pi-SmartEdit uses this in SHA-256 freshness checks. If a non-canonical path (via `path.resolve`/`path.join` only, without symlink resolution) leaks into an envelope, SmartEdit's evidence validation may reject valid reads or, worse, authorize edits against the wrong file (TOCTOU). The reference pattern is in `src/ids.ts:57-62` (`canonicalizeWorkspaceRoot`). Each producer is responsible for following this contract; consumers inherit correct behavior from `validateInspectionEnvelope`.

### Coverage types and patch authorization
- `full-file` → strong evidence; patch can proceed with SHA-256 freshness only.
- `line-range` → strong evidence for the covered range; patch validates via `allowedRanges`.
- `search-match` / `metadata-only` → weak evidence; SmartEdit MUST re-read the file before editing.
- Directory-mode inspect returns `map` mode with zero resources — no file authorization is implied.

### Patch validation
`validatePatchRequest` (`src/contract.ts:130-161`) now requires each edit to carry at least one of `oldText` or `newText`. Empty/missing-both edits are rejected at the protocol boundary. `evidenceRef.inspectionId` must be HEX64 (64-char hex sha256). RPC server (`src/rpc.ts`) provides in-flight requestId dedup; timeouts, cancellation, and disposal are handled.

### NUL-byte path handling
`validateLineRange` and resource validators already reject NUL bytes in `canonicalPath` — no file that hits SmartEdit's SHA-256 check can have an injected NUL path.

### No filesystem dependency in pure functions
`sha256OfString`, `sha256OfBytes`, `resourceIdFor`, `inspectionIdFor`, `hashSessionFilePath` are side-effect-free. Only `canonicalizeWorkspaceRoot` performs sync filesystem I/O (`fs.realpathSync`).

## Residual Risks
- `PatchDetails` / `LifecycleChecks` / `CheckRecord` types are defined in `src/types.ts` but have **no runtime validators** — consumers must validate these at their own boundary.
- `RPC_CHANNELS` constant is exported but has no dedicated test — consumers should import it, not hardcode the channel string. `RPC_CHANNELS.languageIntelligence` and `LANGUAGE_INTELLIGENCE_RPC_METHODS` added in `0.4.0`.
- Language-intelligence validators live in `src/language-intelligence.ts` and are re-exported from the entry point.

---
> Source: [rhinos0608/Pi-Workspace-Protocol](https://github.com/rhinos0608/Pi-Workspace-Protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->

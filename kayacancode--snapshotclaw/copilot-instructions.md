## snapshotclaw

> SnapshotClaw is a content-addressed versioning system for AI agent state (specifically OpenClaw), inspired by AT Protocol's personal data repositories. Read `SPEC.md` for the full architecture.

# CLAUDE.md — SnapshotClaw Development Guide

## What is this project?

SnapshotClaw is a content-addressed versioning system for AI agent state (specifically OpenClaw), inspired by AT Protocol's personal data repositories. Read `SPEC.md` for the full architecture.

## Key concepts

- **MST (Merkle Search Tree)**: Deterministic tree mapping paths → CIDs. Same spec as AT Protocol. Given the same records, always produces the same root hash.
- **CID**: Content identifier — SHA-256 hash of CBOR-encoded data. Using CIDv1 with dag-cbor codec.
- **Commit**: Signed object pointing to an MST root. Contains rev (timestamp-based), prev (chain), Ed25519 signature.
- **Record**: A CBOR-encoded file with path, content, and metadata. Sensitive records are encrypted (AES-256-GCM).
- **CAR file**: Content ARchive — portable single-file export of a snapshot (commit + all reachable blocks).
- **Block store**: Local CID-addressed file storage. Blocks sharded by CID prefix.

## Tech stack

- **Language**: TypeScript (Node.js, ESM)
- **Key deps**: `@ipld/dag-cbor`, `multiformats`, `@noble/ed25519`, `@noble/hashes`, `@ipld/car`, `commander`
- **Test**: vitest
- **Build**: tsup or tsc

## Project structure

```
src/
  core/
    mst.ts           # Merkle Search Tree implementation
    cid.ts           # CID generation utilities
    commit.ts        # Commit creation, signing, verification
    record.ts        # File → Record conversion
    blockstore.ts    # CID-addressed block storage
  crypto/
    signing.ts       # Ed25519 key management + signing
    encryption.ts    # AES-256-GCM encryption for sensitive records
  car/
    export.ts        # CAR file generation (full + diff)
    import.ts        # CAR file parsing + verification
  scanner/
    walker.ts        # File system walker for OpenClaw state
    collections.ts   # Collection type definitions + classification
  cli/
    index.ts         # CLI entry point (commander)
    commands/        # Individual command implementations
      init.ts
      create.ts
      list.ts
      show.ts
      diff.ts
      restore.ts
      export.ts
      import.ts
      sync.ts
      gc.ts
      verify.ts
      log.ts
      keys.ts
      config.ts
  utils/
    tid.ts           # TID (timestamp ID) generation (AT Proto format)
    paths.ts         # Path utilities + defaults
    display.ts       # CLI output formatting
tests/
  core/
    mst.test.ts
    cid.test.ts
    commit.test.ts
    record.test.ts
    blockstore.test.ts
  crypto/
    signing.test.ts
    encryption.test.ts
  car/
    export.test.ts
    import.test.ts
  integration/
    snapshot-lifecycle.test.ts
    diff.test.ts
```

## Implementation order (phases)

### Phase 1: Core MVP
1. `cid.ts` — CID generation from CBOR-encoded data
2. `record.ts` — Convert files to CBOR records with metadata
3. `blockstore.ts` — Store/retrieve blocks by CID
4. `mst.ts` — Full MST: insert, lookup, walk, root hash, key depth
5. `commit.ts` — Create unsigned commits (signing comes in Phase 3)
6. `walker.ts` + `collections.ts` — Scan OpenClaw directories, classify files
7. CLI commands: `init`, `create`, `list`, `show`, `diff`
8. `restore.ts` — Hydrate files from a commit

### Phase 2: Export/Import
1. `car/export.ts` — Full CAR generation
2. `car/import.ts` — CAR parsing + block ingestion
3. Diff-based CAR export
4. CLI commands: `export`, `import`

### Phase 3: Security
1. `signing.ts` — Ed25519 keypair gen, commit signing, verification
2. `encryption.ts` — AES-256-GCM + Argon2id for sensitive records
3. CLI commands: `keys`, `verify`

### Phase 4: Sync + Polish
1. Local directory sync
2. S3 sync
3. Garbage collection
4. OpenClaw hook integration

## MST implementation notes

Follow the AT Protocol MST spec exactly:
- SHA-256 hash the key bytes
- Count leading zero BITS (not bytes), divide by 2, round down = depth
- Fanout of 4
- Key prefix compression within nodes (elide common prefix bytes with previous entry)
- Empty repo = single node with empty entries array
- Prune empty nodes from top and bottom, keep empty intermediate nodes
- Structure is deterministic — same keys/values = same root CID always

Reference implementations:
- https://github.com/DavidBuchanan314/merkle-search-tree (Python, simple)
- https://github.com/bluesky-social/atproto (TypeScript, production)

## Important constraints

- **No plaintext secrets in block store** — encrypt BEFORE computing CID
- **Deterministic** — same input must always produce same output (canonical CBOR, sorted keys)
- **CIDv1 only** — dag-cbor codec (0x71) for structured data, raw (0x55) for file content
- **SHA-256 only** — for both CIDs and MST key depth calculation
- **CBOR must be canonical** — use dag-cbor which enforces deterministic encoding

---
> Source: [kayacancode/snapshotclaw](https://github.com/kayacancode/snapshotclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->

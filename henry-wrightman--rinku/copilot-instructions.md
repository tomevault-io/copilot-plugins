## rinku-protocol

> Rinku protocol conventions — dual Rust/TS impl, proof formats, versioning


# Rinku Protocol

## Dual implementation

Protocol primitives exist in both Rust and TypeScript. Changes to shared semantics (tx hashing, merkle roots, checkpoint format, BLS aggregation) must touch both:

- Rust: `packages/rinku-core/src/`
- TypeScript: `packages/core/src/`

Run conformance tests after changes:
```bash
npm run test -w @rinku/core -- src/__tests__/appendix-g-conformance.test.ts
cargo test -p rinku-core
```

## Consensus invariants (never break silently)

- Checkpoint quorum: 66.6% stake-weighted BLS signatures
- Merge must preserve balance conservation (see `merge/proptests.rs`)
- Partition healing is deterministic — same inputs → same resolution
- `GENESIS_VALIDATORS` format: `address:blsPublicKey;address:blsPublicKey`

## Proof profiles

| Profile | Use | Package |
|---------|-----|---------|
| A | Merkle inclusion | `packages/core/src/compact-proof.ts` |
| B | Checkpoint-attested | `packages/core/src/receipt.ts` |
| C | Self-contained (offline) | `packages/core/src/self-proof.ts` |

## Versioning

Protocol upgrades follow `docs/VERSIONING.md`. Bump `protocolVersion` only with migration path documented.

---
> Source: [henry-wrightman/rinku](https://github.com/henry-wrightman/rinku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

## rust-consensus

> Rust consensus, merge, and slashing code standards


# Rust Consensus Code

## Module map

| Module | Responsibility |
|--------|----------------|
| `consensus.rs` | Vote accumulator, BLS quorum, finality |
| `checkpoint.rs` | Checkpoint creation and attestation |
| `merge/` | Partition merge: conflict detection, cascade, resolution |
| `slashing.rs` | Double-sign and liveness evidence |
| `fast_path.rs` | Provisional acceptance (non-final) |
| `gossip.rs` | libp2p sync, bloom filters, checkpoint propagation |

## Required for changes

1. Add or extend `#[test]` / `proptest` in the same module
2. For merge logic: also check `tests/merge_e2e.rs`
3. For P2P behavior: check `tests/p2p_integration.rs`
4. Run `cargo clippy --all-targets --workspace -- -D warnings`

## Patterns

```rust
// ✅ Use explicit error types
return Err(ConsensusError::QuorumNotReached { have, need });

// ❌ Don't swallow consensus failures
let _ = vote_accumulator.try_finalize();
```

## Security-sensitive areas

- `validator_identity.rs` — key persistence
- `sync_verification.rs` — snapshot merkle verification
- `wasm_runtime.rs` — fuel limits, import rejection

Never log private keys or `VALIDATOR_KEY_PASSWORD` values.

---
> Source: [henry-wrightman/rinku](https://github.com/henry-wrightman/rinku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

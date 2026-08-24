## morpheum-signing

> <!-- morpheum-claude-framework v2026-08-07 — shared blocks synced by sync.sh; edit prose freely -->

<!-- morpheum-claude-framework v2026-08-07 — shared blocks synced by sync.sh; edit prose freely -->
# morpheum-signing

Universal multi-chain signing SDK for the Morpheum L1: one fluent `TxBuilder` producing
exact `Tx`/`SignDoc`/`TxRaw` bytes across native ed25519, EVM (secp256k1), Solana
(ed25519), and Bitcoin Taproot (BIP-340), plus TradingKey/VC delegation for autonomous
agents. `no_std` core; all secret material is `ZeroizeOnDrop`.

**This repo is PUBLIC on GitHub.** Everything committed here is public content. Do not add
internal architecture detail about private sibling repos beyond their names and build
requirements.

## Layout

- `crates/core` — `no_std` heart: builder, preimage construction, claims, nonce, verifier
- `crates/native` — adapters/providers/signers for native targets
- `crates/wasm-lib` — shared wallet adapters reused by wasm builds
- `crates/wasm` — wasm-bindgen package + TS bindings (published surface for TS consumers)
- `fuzz/` — seed generation, claim construction/encoding, address mapping targets (nightly)

## Commands `[host]`

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
cargo clippy --target wasm32-unknown-unknown -p morpheum-signing-wasm --lib -- -D warnings
wasm-pack build crates/wasm --target nodejs --out-dir pkg-node   # the package TS consumes
```

This repo builds on the host (pure Rust). Requires sibling checkouts (see ripple below).

## Invariants

- **The two signing entry points are deliberately NOT merged**: `TxBuilder::sign()`
  (connected signer, derives `SignerInfo`, embeds claims, sets a deadline) vs
  `build_sign_doc()` (signer-less, raw public key, explicit chain type). They assemble
  different `SignerInfo`s for real reasons and already share the canonical construction —
  do not "DRY" them without new evidence.
- **The genesis guard in `TxBuilder::sign()` is fail-closed and runs before nonce
  resolution.** Provider nonces are monotonic and not returned on failure; refusing after
  fetching would burn a nonce and leave an unskippable gap. Both orderings return the same
  error, so the ordering is pinned by a counting-nonce-provider test
  (`the_refusal_does_not_burn_a_monotonic_nonce`) — keep that test meaningful.
- **The golden preimage vector pins bytes, not code.** Its constants were captured from the
  previously shipped package, so they prove refactors preserve the preimage rather than
  pinning the code against itself. They are single unbroken literals — never re-wrap them
  (a wrapped literal once silently gained a byte) and never regenerate them from current
  code.
- **Never hand-write TS declarations for symbols wasm-bindgen generates.** TypeScript
  merges duplicate declarations, and consumer-side type pins can't detect a reintroduced
  overload (`Parameters<T>` reads only the last one). The `typescript_custom_section`
  declares interfaces only; the surface is pinned here by `crates/wasm/tests/`.
- `nonce` is required **and returned** by the wasm API so signed-vs-sent divergence is
  unrepresentable — keep that shape.
- The wasm32 clippy leg is part of this repo's own gate: it catches arity/signature breaks
  that only `wasm-pack build` would otherwise surface.

<!-- framework:begin ripple -->
## Cross-repo ripple

- Depends on siblings: `../morpheum-primitives` (full-protos), three `../cryptogram` crates.
- Dependents: `mormcore` (crates/core), `morpheum-sdk` (core + native + wasm-lib), and
  `morpheum-sdk-ts` via the wasm package
  (`wasm-pack build crates/wasm --target nodejs --out-dir pkg-node`).
- The TS type surface is pinned HERE (crates/wasm tests), never at consumers — downstream
  type pins cannot detect a reintroduced duplicate declaration.
- After any change to crates/core or crates/wasm: rebuild pkg-node and typecheck
  `../morpheum-sdk-ts` (`npx tsc --noEmit`).
<!-- framework:end ripple -->

## Verification

- CI = fmt + clippy (native and wasm32 legs) + `cargo test --workspace --all-features`.
  Run all four locally or say which you skipped.
- Fuzz targets live in their own nightly workspace under `fuzz/` — run when touching seed
  generation, claim encoding, or address mapping.
- `.claude/rules/signing-invariants.md` restates the invariants above for path-scoped
  review.

---
> Source: [morpheum-labs/morpheum-signing](https://github.com/morpheum-labs/morpheum-signing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

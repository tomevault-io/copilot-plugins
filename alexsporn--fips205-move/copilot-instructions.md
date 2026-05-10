## fips205-move

> Pure Move implementation of **FIPS 205 SLH-DSA** signature verification for the IOTA blockchain. Verification-only (no signing/keygen). Post-quantum secure, based solely on SHA-256.

# CLAUDE.md -- Project Context for fips205-move

## What This Project Is

Pure Move implementation of **FIPS 205 SLH-DSA** signature verification for the IOTA blockchain. Verification-only (no signing/keygen). Post-quantum secure, based solely on SHA-256.

Supports **SHA2-128s** and **SHA2-128f** parameter sets. Generic architecture designed for future 192/256-bit variants once SHA-512 native support is available.

## Build & Test

```bash
iota move build
iota move test -i 100000000000    # high gas limit required (many SHA-256 calls)
```

## Architecture

Generic parameterized design -- all algorithm modules receive `&Params`.

### Verification layers (FIPS 205 Section 10)

- **Algorithm 20** (`verify` / `verify_with_context`): Public API. Constructs `M' = toByte(0,1) || toByte(|ctx|,1) || ctx || M`, then calls Algorithm 22. The `0x00` first byte is for "pure" mode (vs `0x01` for pre-hash mode, not implemented).
- **Algorithm 22** (`verify_internal`): Core cryptographic algorithm. Takes the already-wrapped message M' and runs H_msg -> FORS -> hypertree verification.

### Module structure

- `params.move` -- `Params` struct + constructors: `sha2_128s()`, `sha2_128f()`
- `slh_dsa_sha2_128s.move` -- public entry: `verify(msg, sig, pk)` and `verify_with_context(msg, sig, pk, ctx)`
- `slh_dsa_sha2_128f.move` -- public entry: `verify(msg, sig, pk)` and `verify_with_context(msg, sig, pk, ctx)`
- `slh_dsa.move` -- core: `verify(msg, sig, pk, ctx, params)` (Algorithm 20) and `verify_internal(msg, sig, pk, params)` (Algorithm 22)
- `fors.move` -- FORS forest, Algorithm 17
- `ht.move` -- hypertree, Algorithm 13
- `xmss.move` -- XMSS tree, Algorithm 11
- `wots.move` -- WOTS+ chains, Algorithms 5 & 8
- `thash.move` -- SHA-256 tweakable hashes: F, H, T_l, H_msg with MGF1 (SHA-256 specific)
- `adrs.move` -- 32-byte ADRS domain separation, 22-byte compressed form
- `utils.move` -- slice, to_int, to_byte, base_2b

### Test modules (64 tests total)

- `*_128s_tests.move` / `*_128f_tests.move` -- reference crate vectors (empty context)
- `*_128s_acvp_tests.move` / `*_128f_acvp_tests.move` -- NIST ACVP `signatureInterface: internal` (Algorithm 22, no context wrapping)
- `*_128s_acvp_ctx_tests.move` / `*_128f_acvp_ctx_tests.move` -- NIST ACVP `signatureInterface: external, preHash: pure` (Algorithm 20 with non-empty context strings)

## NIST ACVP Test Vector Notes

Source: `ACVP-Server/gen-val/json-files/SLH-DSA-sigVer-FIPS205/internalProjection.json`

The ACVP sigVer file has three `signatureInterface` / `preHash` combinations per parameter set:
- **`internal/none`** -- tests Algorithm 22 directly (raw message, no wrapping)
- **`external/pure`** -- tests Algorithm 20 (message + context string, `0x00` prefix)
- **`external/preHash`** -- tests pre-hash mode (`0x01` prefix + OID + hash of message) -- **not implemented**, only useful for signing large data without full buffering

## Adding a New Parameter Set

1. Add a constructor in `params.move` (e.g., `sha2_192s()`)
2. Create a thin entry-point module (e.g., `slh_dsa_sha2_192s.move`)
3. For SHA-512 variants: create `thash_sha512.move` with 128-byte block padding

## Key Constants

| Param | 128s | 128f |
|-------|------|------|
| n | 16 | 16 |
| h | 63 | 66 |
| d | 7 | 22 |
| h' | 9 | 3 |
| a | 12 | 6 |
| k | 14 | 33 |
| m | 30 | 34 |
| sig | 7,856 | 17,088 |

## Important Implementation Details

- **Context support**: `verify_with_context` accepts 0-255 byte context strings per FIPS 205 Section 10.2
- **MGF1 generalized**: h_msg loops for any m (1 iteration for m=30, 2 for m=34)
- **No index assignment**: use `*v.borrow_mut(i) = x` instead of `v[i] = x`
- **No explicit Iota dep needed**: IOTA CLI auto-adds framework dependencies
- **Entry points are `public`**, core functions are `public(package)`
- **Params struct fields**: private, accessed via accessor functions
- **thash.move is SHA-256 specific**: future SHA-512 variants need a parallel module

## Reference

- FIPS 205: https://csrc.nist.gov/pubs/fips/205/final
- Rust reference: https://github.com/integritychain/fips205
- NIST ACVP vectors: https://github.com/usnistgov/ACVP-Server

---
> Source: [alexsporn/fips205-move](https://github.com/alexsporn/fips205-move) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->

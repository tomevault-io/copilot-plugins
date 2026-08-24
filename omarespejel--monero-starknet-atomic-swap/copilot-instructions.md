## rust-crypto

> Rust cryptography security rules


# Rust Cryptography Rules

## Security Requirements

- All `Scalar` types MUST derive `Zeroize, ZeroizeOnDrop`
- All hash functions MUST have domain separation prefix
- All public functions MUST validate inputs (check != 0)
- NEVER use `format!("{:?}", secret)` or similar debug output for secrets

## Required Imports

```rust
use zeroize::{Zeroize, ZeroizeOnDrop};
use curve25519_dalek::scalar::Scalar;
use curve25519_dalek::constants::ED25519_BASEPOINT_POINT;
```

## Nonce Generation Pattern

```rust
fn generate_nonce(secret: &Scalar, context: &[u8]) -> Result<Scalar, DleqError> {
    let mut hasher = Sha256::new();
    hasher.update(b"DOMAIN_PREFIX_V1");  // REQUIRED
    hasher.update(secret.to_bytes());
    hasher.update(context);
    // ... validate result != 0
    // ... retry if zero
}
```

## Error Handling

- Return `Result<T, CryptoError>` not `T`
- Define explicit error types for crypto failures
- Never panic on invalid inputs, return errors

## Testing

- Use proptest for property-based tests
- Test edge cases: zero, max scalar, identity point
- Test input validation: wrong points, wrong hashlocks

---
> Source: [omarespejel/monero-starknet-atomic-swap](https://github.com/omarespejel/monero-starknet-atomic-swap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

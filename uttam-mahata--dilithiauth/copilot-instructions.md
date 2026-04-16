## dilithiauth

> DilithiAuth is a research-oriented Identity Provider (IdP) designed to mitigate the **"Harvest Now, Decrypt Later" (HNDL)** threat. It implements the Authorization Code Flow using hybrid signatures that combine classical **Ed25519** with NIST-standardized **ML-DSA-44** (FIPS 204).

# DilithiAuth: Post-Quantum Identity Provider

DilithiAuth is a research-oriented Identity Provider (IdP) designed to mitigate the **"Harvest Now, Decrypt Later" (HNDL)** threat. It implements the Authorization Code Flow using hybrid signatures that combine classical **Ed25519** with NIST-standardized **ML-DSA-44** (FIPS 204).

## Project Overview

- **Goal:** Provide a quantum-safe identity solution while addressing the "token bloat" problem caused by large ML-DSA signatures.
- **Key Technology:** [ML-DSA-44](https://csrc.nist.gov/pubs/fips/204/final) (formerly Dilithium), part of the NIST Post-Quantum Cryptography (PQC) standards.
- **Architecture:**
    - `pkg/signer`: Implements the hybrid signature scheme (ML-DSA-44 + Ed25519).
    - `pkg/oauth`: Handles OIDC Discovery, JWKS, and token issuance (JWT and CBOR Web Tokens).
    - `pkg/transport`: Provides HTTP middleware for large headers and a signature reference sidecar store.
    - `cmd/server`: The main IdP entry point.

### Token Strategies
1. **Thin Tokens:** JWT/CWT contains only the Ed25519 signature and a hash reference (`pqc_ref`) to the full ML-DSA signature stored in a sidecar (e.g., Redis).
2. **Jumbo Tokens:** "Jumbo" JWTs embed the full ~2.4 KB ML-DSA signature directly, resulting in tokens ~4 KB in size.
3. **CWT (CBOR Web Tokens):** Binary tokens (RFC 8392) that are 20-30% smaller than their JWT counterparts.

## Building and Running

### Prerequisites
- Go 1.24 or later.

### Key Commands
- **Build the server:**
  ```bash
  go build -o dilithiauth-server ./cmd/server
  ```
- **Run the server:**
  ```bash
  # Optional environment variables:
  # DILITHIAUTH_ISSUER=https://auth.example.com
  # DILITHIAUTH_ADDR=:8080
  ./dilithiauth-server
  ```
- **Run tests:**
  ```bash
  go test ./...
  ```

## Development Conventions

- **Hybrid Keys:** Always use `signer.HybridKeyPair` to ensure every token is signed by both classical and quantum-safe algorithms.
- **Large Headers:** The server is configured with `MaxHeaderBytes = 16 KiB` to accommodate Jumbo tokens. Ensure any proxy (like Nginx) is similarly configured using `deployments/nginx.conf` as a guide.
- **Testing:** Add tests to the respective `*_test.go` files in `pkg/`. Use `httptest` for endpoint verification.
- **Sidecar Storage:** For production, the in-memory `transport.SigStore` should be replaced with a distributed cache (e.g., Redis) to support horizontal scaling.

## Key Endpoints
- `GET /.well-known/openid-configuration`: OIDC Discovery Document.
- `GET /.well-known/jwks.json`: Public keys (includes `mldsa44_pub` field).
- `POST /token/thin?sub=alice`: Issue a thin JWT.
- `POST /token/full?sub=bob`: Issue a jumbo JWT.
- `POST /token/cwt?sub=charlie`: Issue a CBOR Web Token.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Uttam-Mahata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

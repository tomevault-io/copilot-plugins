## octo-sts-rust

> This file provides context for Claude Code when working on this project.

# CLAUDE.md - Project Context for Claude

This file provides context for Claude Code when working on this project.

## Project Overview

octo-sts-rust is a drop-in replacement for [octo-sts](https://github.com/octo-sts/app), a GitHub Security Token Service. It exchanges OIDC tokens (and PATs) for short-lived GitHub API tokens. It runs on both Cloudflare Workers (WASM) and GCP Cloud Run (native binary), with optional GCP KMS asymmetric signing support.

## Key Documentation

- **Architecture:** `docs/architecture/README.md` - System design, components, data flow
- **Design decisions:** `docs/plans/2026-02-03-octo-sts-rust-design.md` - Original design rationale
- **Multi-platform design:** `docs/plans/2026-02-04-multi-platform-design.md` - Workspace architecture

## Project Structure

```
octo-sts-rust/              # Cargo workspace
├── core/                   # Platform-agnostic business logic
│   └── src/
│       ├── lib.rs          # Crate root
│       ├── config.rs       # Config (domain, webhook secret)
│       ├── error.rs        # Error types and HTTP status mapping
│       ├── platform.rs     # Platform traits: Cache, HttpClient, Clock, Environment, JwtSigner
│       ├── sts/
│       │   ├── exchange.rs     # OIDC token exchange
│       │   ├── exchange_pat.rs # PAT exchange (new feature)
│       │   └── revoke.rs       # Token revocation
│       ├── policy/
│       │   ├── types.rs    # TrustPolicy / OrgTrustPolicy structs
│       │   ├── compile.rs  # Regex compilation and validation
│       │   ├── check.rs    # OIDC claim matching against policies
│       │   └── mod.rs      # Policy loading and caching
│       ├── github/
│       │   ├── auth.rs     # App JWT signing (PemJwtSigner), installation tokens
│       │   ├── api.rs      # GitHub API client (file content, check runs)
│       │   └── webhook.rs  # Webhook signature verification and event handling
│       └── oidc/
│           ├── discovery.rs # OpenID Connect discovery
│           ├── jwks.rs      # JSON Web Key Set fetching
│           └── validate.rs  # Token validation
├── cloudflare/             # Cloudflare Workers adapter
│   ├── src/
│   │   ├── lib.rs          # Worker entry point, request router
│   │   └── platform.rs     # Workers KV cache, Fetch HTTP client, JS clock
│   └── wrangler.toml       # Wrangler config (dev + production environments)
├── gcp/                    # GCP Cloud Run adapter
│   └── src/
│       ├── main.rs         # Hyper HTTP server, signer mode detection
│       ├── platform.rs     # Moka cache, reqwest HTTP client, system clock
│       └── kms.rs          # KmsJwtSigner (Cloud KMS asymmetric signing)
└── terraform/              # Cloudflare infrastructure
```

## Key Concepts

**Trust Policies:** YAML files at `.github/chainguard/{name}.sts.yaml` that define which OIDC tokens can be exchanged for GitHub tokens with what permissions.

**PAT Policies:** YAML files at `.github/chainguard/{name}.pat.yaml` that define which PAT holders (by org membership) can get scoped tokens.

**Token Exchange Flow:** OIDC token -> validate signature -> match policy -> generate GitHub installation token with scoped permissions.

**PAT Exchange Flow:** PAT -> validate against GitHub API -> check org membership -> match PAT policy -> generate scoped installation token.

**Platform Traits:** All core logic depends on abstract traits in `core/src/platform.rs`:
- `Cache` — key-value storage with TTL (Workers KV or Moka)
- `HttpClient` — outbound HTTP (Fetch API or reqwest)
- `Clock` — current time (JS `Date.now()` or `SystemTime`)
- `Environment` — env vars and secrets
- `JwtSigner` — GitHub App JWT signing (`PemJwtSigner` or `KmsJwtSigner`)

**Signer Mode Detection (GCP):** If `KMS_KEY_NAME` env var is set, uses `KmsJwtSigner` (Cloud KMS); otherwise uses `PemJwtSigner` (PEM key in env var).

## Build and Test

```bash
# Core unit tests (59 tests)
cargo test -p octo-sts-core

# WASM build check
cargo check -p octo-sts-cloudflare --target wasm32-unknown-unknown

# GCP native binary
cargo build -p octo-sts-gcp --release

# Cloudflare deploy (dev)
cd cloudflare && wrangler deploy

# Cloudflare deploy (production)
cd cloudflare && wrangler deploy --env production
```

## Important Technical Details

- `async_trait(?Send)` is required on all async traits for WASM compatibility
- `Cache` trait uses raw bytes (`get_bytes`/`put_bytes`) to be dyn-compatible; free functions `cache_get<T>`/`cache_put<T>` handle serialization
- GCP server uses hyper directly (not axum) because axum requires Send futures
- Config holds only `domain` and `github_webhook_secret`; app ID and private key are in the signer
- Org-level policies (scope `owner/.github`) skip repository access checks and use the policy's repository list for token scoping

## Common Tasks

When modifying trust policy logic, check `core/src/policy/`.
When modifying GitHub API interactions, check `core/src/github/`.
When modifying OIDC validation, check `core/src/oidc/`.
When modifying token exchange, check `core/src/sts/`.
When modifying platform adapters, check `cloudflare/src/` or `gcp/src/`.
When modifying KMS signing, check `gcp/src/kms.rs`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Epiphytic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

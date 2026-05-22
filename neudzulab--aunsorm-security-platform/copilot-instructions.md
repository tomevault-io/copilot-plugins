## aunsorm-security-platform

> **Last Updated:** 2026-01-31

# Aunsorm Agent Coordination

**Version:** 0.5.0  
**Last Updated:** 2026-01-31

## Primary Directive

**All development work must align with [PROD_PLAN.md](PROD_PLAN.md).**

This repository is coordinated by specialized domain agents. All new features, refactoring, and infrastructure changes must be tracked as tasks in `PROD_PLAN.md` with checkbox format for progress tracking.

---

## Principles

- All test errors must be fixed — even warnings are unacceptable
- No deprecated dependencies are allowed
- Issues must never be suppressed; instead, they should be resolved properly
- Mocking must not be used as a replacement for actual implementations
- Sealed classes, methods, or structures must remain intact and must not be modified
- All code must compile on MSRV 1.76+
- `unsafe` code is forbidden (`#![forbid(unsafe_code)]`)

---

## Agent Responsibilities

### Crypto Agent
- **Scope:** `crates/core`, `crates/pqc`, `crates/packet`
- **Focus:** Cryptographic primitives, PQC implementations, Native RNG compliance
- **Current Priority:** Complete PQC security audit and NIST compliance validation

### Platform Agent
- **Scope:** `crates/server`, `crates/cli`, `crates/wasm`, Docker/Kubernetes manifests
- **Focus:** Microservice orchestration, API gateway, deployment automation
- **Current Priority:** Kubernetes migration and production infrastructure setup

### Identity Agent
- **Scope:** `crates/jwt`, `crates/x509`, `crates/kms`, `crates/acme`, `crates/mdm`
- **Focus:** Authentication, certificates, key management, device enrollment
- **Current Priority:** HSM integration for KMS and OAuth 2.0 complete implementation

### Interop Agent
- **Scope:** `benches/`, `fuzz/`, `tests/`, `examples/`, CI/CD pipelines
- **Focus:** Testing, benchmarking, security audits, documentation
- **Current Priority:** Achieve >80% test coverage and third-party security audit

---

## 🎲 Native RNG Mandatory (v0.4.5+)

**CRITICAL:** All cryptographic random number generation MUST use `AunsormNativeRng`.

### Forbidden Usage:
- ❌ Direct `OsRng` usage (except initial entropy seeding)
- ❌ HTTP `/random/number` endpoint calls (6.4s overhead)
- ❌ `rand::thread_rng()` or other stdlib RNGs
- ❌ `ChaCha8Rng` or other external RNG implementations (except in tests)

### Required Usage:
- ✅ `AunsormNativeRng` - Same implementation across all crates
- ✅ HKDF + NEUDZ-PCS + AACM mixing - Same algorithm as server
- ✅ 4x Performance - Native vs HTTP (1.5s vs 6.4s RSA-2048)

### Implementation Pattern:

```rust
// ✅ CORRECT - Same pattern in every crate
use crate::rng::AunsormNativeRng;

pub fn generate_key() -> Result<Key, Error> {
    let mut rng = AunsormNativeRng::new();
    Key::generate_with_rng(&mut rng)
}

// ❌ WRONG - Now forbidden
use rand_core::OsRng;

pub fn generate_key() -> Result<Key, Error> {
    let mut rng = OsRng;  // FORBIDDEN!
    Key::generate_with_rng(&mut rng)
}
```

### Crate-Specific Requirements:
- **ACME**: Ed25519, P256, RSA account keys → `AunsormNativeRng`
- **JWT**: Ed25519 signing keys, JTI generation → `AunsormNativeRng`
- **KMS**: AES-GCM nonce generation → `AunsormNativeRng`
- **X509**: RSA key generation for certificates → `AunsormNativeRng`
- **NEW CRATES**: Must create own `src/rng.rs` module

---

## Critical Rules

### 1. Code Quality Gates

Every commit must pass:

```bash
cargo fmt --all
cargo clippy --all-targets --all-features
cargo test --all-features
cargo deny check
```

### 2. Production Plan Compliance

- All new work items must be added to `PROD_PLAN.md` with `[ ]` checkbox
- Mark tasks as `[x]` only when fully completed and tested
- Do NOT modify completed tasks - create new revision tasks instead
- Each PR must reference its `PROD_PLAN.md` task

### 3. Security Requirements

- **No `unsafe` code** - `#![forbid(unsafe_code)]` enforced
- **MSRV 1.76+** - Minimum Supported Rust Version
- **Dependency audits** - `cargo audit` must be clean
- **Fuzz testing** - All parsers/decoders must have fuzz targets

### 4. Documentation Standards

- Update `README.md` for any new endpoints or services
- Update `port-map.yaml` for any port changes
- Add `CHANGELOG.md` entry for version changes
- Technical architecture changes require `PROJECT_SUMMARY.md` updates

### 5. Revision Lock Policy

Items marked as `[x]` (completed) in README, PROD_PLAN.md, or TODO.md are locked:
- Do not reopen completed items
- Create a new item with `Revize:` prefix if changes are needed
- Reference the original item in the revision

---

## JWT Response Structure - SEALED (v0.5.0+)

**⚠️ SEALED STRUCTURE - DO NOT MODIFY WITHOUT SECURITY REVIEW ⚠️**

The JWT verification response structure is **sealed** to prevent duplicate field serialization errors and maintain client compatibility.

**Canonical Structure (`crates/server/src/routes.rs`):**

```rust
#[derive(Serialize)]
pub struct JwtPayload {
    pub subject: String,
    pub audience: String,
    pub issuer: String,
    pub expiration: u64,
    #[serde(rename = "issuedAt", skip_serializing_if = "Option::is_none")]
    pub issued_at: Option<u64>,
    #[serde(rename = "notBefore", skip_serializing_if = "Option::is_none")]
    pub not_before: Option<u64>,
    #[serde(rename = "relatedId", skip_serializing_if = "Option::is_none")]
    pub related_id: Option<String>,
    #[serde(rename = "jwtId", skip_serializing_if = "Option::is_none")]
    pub jwt_id: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub extras: Option<serde_json::Map<String, serde_json::Value>>,
}
```

**Critical Rules:**
1. ❌ DO NOT flatten `extras` - causes duplicate field errors
2. ❌ DO NOT add raw claims to top-level
3. ✅ Standard JWT claims use canonical names at top-level
4. ✅ Custom claims MUST be nested under `extras`
5. ✅ Use camelCase for JSON fields

**Modification requires:** Security review, client compatibility assessment, API version bump, CHANGELOG update.

---

## Servis Ağacı Güncelleme Direktifi

**YENİ ÖZELLİK/ENDPOINT EKLENDİĞİNDE MUTLAKA YAPILACAKLAR:**

### 1. README.md Server Endpoint Ağacını Güncelle
- Yeni endpoint eklendiğinde `README.md` içindeki endpoint ağacına ekle
- Yarım/tamamlanmamış özellik bile olsa `[Planlandı v0.X.0]` veya `[Devam Ediyor]` işaretiyle ekle

### 2. Servis Durumu İşaretleri
- ✅ Aktif/Çalışıyor: Endpoint tamamen çalışıyor ve test edilmiş
- 🚧 Geliştirme: Kod var ama endpoint route'u henüz eklenmedi
- 📋 Planlandı: Crate var, servis entegrasyonu bekliyor
- 🔮 Gelecek: Henüz tasarım aşamasında

### 3. Port Mapping Güncelleme (`port-map.yaml`)
- Yeni port ekleme/değiştirme durumunda `port-map.yaml` güncelle
- **Aunsorm portları: 50010-50023** (mevcut)
- **Zasian Media Platform portları: 50030-50037** (v0.6.0)

### 4. Environment Değişkenleri (`.env`)
- **Production Override**: `HOST` environment variable ile localhost aşılabilir
- ❌ **Yasak**: Kod içine localhost/127.0.0.1 hardcode yazmak
- ✅ **Doğru**: Environment variable + fallback pattern kullanmak

### 5. OpenAPI Dokümantasyon Güncelleme (`openapi/`)
- Yeni endpoint eklendiğinde ilgili `{service}-service.yaml` dosyasını güncelle
- Request/response schema'ları, örnek payloadlar ve hata kodları ekle

---

## Workflow

1. **Check `PROD_PLAN.md`** - Find unassigned tasks in your domain
2. **Create branch** - Use format: `agent/crypto/task-description`
3. **Implement** - Follow code quality gates
4. **Test** - Unit, integration, and manual testing required
5. **Document** - Update README, port-map, OpenAPI specs
6. **PR Review** - Requires approval from domain agent lead
7. **Merge** - Update `PROD_PLAN.md` checkbox `[ ]` → `[x]`

### Sorumluluk Matrisi

| Agent | Primary Ownership |
|-------|------------------|
| **Platform Agent** | Server endpoint ağacı, deployment |
| **Crypto Agent** | Core, PQC, Packet servisleri |
| **Identity Agent** | JWT, X509, KMS, ID servisleri |
| **Interop Agent** | Test/benchmark, eksik servis tespiti |

---

## Communication

- **Questions:** Open GitHub issue with `[Agent Question]` prefix
- **Blockers:** Tag `@platform-lead` in issue
- **Security:** See `SECURITY.md` for disclosure process
- **Urgent:** Tag `@all-agents` in issue (use sparingly)

---

## Versioning

Current version: **0.5.0**

- **Patch (0.5.x):** Bug fixes, documentation updates
- **Minor (0.x.0):** New features, backward-compatible API changes
- **Major (x.0.0):** Breaking changes (target: 1.0.0)

All version bumps require:
1. Update all `Cargo.toml` files
2. Update `CHANGELOG.md`
3. Update version references in documentation
4. Tag release in git: `v0.5.0`

---

## Production Readiness

**See [PROD_PLAN.md](PROD_PLAN.md) for complete production deployment checklist.**

### Current Blockers for v1.0.0:
- [ ] Clock attestation production NTP server deployment
- [ ] PostgreSQL migration from SQLite
- [ ] Kubernetes manifests and Helm charts
- [ ] Third-party security audit completion
- [ ] HSM integration for key management

---

## Contact

For urgent production issues or security concerns, see `SECURITY.md`.

---
> Source: [Neudzulab/Aunsorm-Security-Platform](https://github.com/Neudzulab/Aunsorm-Security-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->

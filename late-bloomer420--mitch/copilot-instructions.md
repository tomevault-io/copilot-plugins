## mitch

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

AskMI is a privacy-preserving proof mediation middleware ("The Forgetting Layer") — ZK-style credential verification with crypto-shredding, fail-closed policy engine, GDPR Art. 25 + eIDAS 2.0 / EUDI compatible. TypeScript monorepo with pnpm workspaces.

## Commands

```bash
# Install dependencies
pnpm install

# Build all packages (respects dependency graph)
pnpm build

# Run all tests (latest QA evidence: 46 turbo tasks, 1820+ individual tests)

pnpm test

# Run tests for a single package
pnpm --filter @askmi/policy-engine test

# Run a single test file
cd src/packages/policy-engine && npx vitest run src/__tests__/engine.test.ts

# Lint (0 errors, 0 warnings)
pnpm lint

# Format
pnpm format
```

**Turbo v2** is used — config uses `tasks` in `turbo.json`.

## Architecture

**Monorepo layout:** `src/packages/` (29 packages) + `src/apps/` (3 apps)

### Core packages

- **policy-engine** — The central "Privacy Firewall" / ZKQF. Evaluates disclosure requests → ALLOW/DENY/PROMPT verdicts. Contains: engine.ts (main evaluator), kpi.ts, rate-limiter.ts, proof-fatigue.ts, jurisdiction.ts, config-profiles.ts, allow-assertion.ts
- **shared-crypto** — `@askmi/shared-crypto`. All crypto primitives: key generation, signing (Ed25519/P-256), encryption (AES-256-GCM), JWE, WebAuthn, PQC (ML-DSA, ML-KEM via @noble/post-quantum), crypto-agility negotiation, pairwise DIDs, DID quorum resolution
- **shared-types** — `@askmi/shared-types`. Central type definitions shared across all packages
- **layer-resolver** — Resolves protection layers (WELT/GRUNDVERSORGUNG/VULNERABLE) and the neutral per-claim sensitivity view. Two maps: enforcement `LAYER_MAP` (`getMinimumLayerForData`) and a visibility-only `VISIBILITY_LAYER_MAP` superset (`resolveLayerForData`/`sensitivityForData`); extending the enforcement map changes policy verdicts, so visibility vocabulary stays in the superset only

### Protocol packages

- **oid4vci** — OpenID for Verifiable Credential Issuance (wallet-side)
- **oid4vp** — OID4VP 1.0 wallet-side (presentation-request, vp-token, response-builder)
- **oid4vp-verifier** — OID4VP verifier-side (request-builder, response-verifier)
- **mdoc** — ISO 18013-5 mDL/mdoc: CBOR codec, COSE Sign1, mdoc types
- **verifier-sdk** / **verifier-browser** — Verifier integration libraries

### Infrastructure packages

- **anchor-service** — Merkle batch anchoring + L2 provider stubs
- **revocation-statuslist** — `@askmi/revocation-statuslist`. StatusList2021, multi-source resolver
- **secure-storage** — IndexedDB-backed encrypted storage (uses fake-indexeddb in tests)
- **secure-memory** — Memory-safe credential handling
- **audit-log** — Immutable audit trail
- **webauthn-verifier** — WebAuthn + step-up authentication
- **predicates** — Predicate proof definitions (age-over, range, set-membership)
- **wallet-core** — Core wallet logic (~700 LOC WalletService, planned for decomposition)

### Apps

- **wallet-pwa** — React 18 + Vite PWA on port 5174 (jsdom test env, needs IndexedDB + elementFromPoint mocks)
- **issuer-mock** — Mock credential issuer on port 3005
- **verifier-demo** — Demo verifier on port 3004 + frontend

## Key Conventions

- **Fail-closed principle:** Ambiguous policy evaluations → DENY. Never default to ALLOW.
- **DecisionCapsule fields:** `verdict`, `decision_id`, `policy_hash` (NOT `policy_manifest_id`)
- **Conventional commits:** `feat:`, `fix:`, `docs:`, `test:`, `chore:`
- **Code style:** Prettier (single quotes, 2-space indent, trailing commas es5, 100 print width). ESLint with `@typescript-eslint/no-explicit-any: warn`, unused vars prefixed with `_`.
- **Test framework:** Vitest. Some packages use `environment: 'node'`, wallet-pwa uses `environment: 'jsdom'` with setup files for IndexedDB mocking.
- **No breaking changes** to public package APIs without explicit approval.
- **policy-engine index.ts** has many exports — check for naming conflicts when adding new modules.
- **npm scope reality:** AskMI is the active project/package brand. All workspace packages use the `@askmi/*` scope; new packages must stay on that scope.
- **Repo truth sources:** Read `docs/DOCS_CANON.md` before updating status, backlog, QA evidence, session history, or agent-facing instructions.

## Testing Notes

- Tests include `src/__tests__/` directories within each package
- wallet-pwa tests require setup mocks: `fake-indexeddb/auto`, `document.elementFromPoint` stub, `getAll`/`getAllKeys`/`clear` for SecureStorage
- secure-storage tests use `fake-indexeddb/auto` via setup file
- Run `ASKMI_TEST_MODE=1` for test-mode-specific behavior

## CI

GitHub Actions (`.github/workflows/ci.yml`): build → test → lint on Node 22 + pnpm 9. Separate security audit job runs `pnpm audit`. Layer validation job tests policy-engine E2E scenario.

---
> Source: [Late-bloomer420/miTch](https://github.com/Late-bloomer420/miTch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

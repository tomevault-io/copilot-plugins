## attestrack

> **Repository:** `github.com/matt-cochran/attestrack` (currently private, pre-launch)

# Attestrack (Public Repository)

**Repository:** `github.com/matt-cochran/attestrack` (currently private, pre-launch)
**License:** MIT
**Purpose:** Attestrack — self-hosted analytics, server-side measurement, and **community consent** (`consent-js` + KV `ConsentConfig` + mandatory Worker strategies, per ADR-010); attorney-maintained regulation and Proof are licensed Attestrue extensions
**Last updated:** 2026-07-25

See `CLAUDE.md` for the full, authoritative repo guide (structure, dependency graph, CI). Highlights below.

## Commands

```bash
pnpm install
pnpm build                  # Turborepo build
pnpm test                   # Vitest unit + contract tests
pnpm typecheck
pnpm lint                   # ESLint flat config, repo root
pnpm size-check             # consent-js 12 kB budget
pnpm boundary-check         # ADR-009: forbidden paths + licensed-sibling refs
pnpm route-contract-check   # docs/oss-http-contract.json vs worker-core source
```

**Requirements:** Node >= 20, pnpm 9

## Packages (all `0.0.0`, nothing published to npm yet)

`types`, `schema` (+ `warehouse/clickhouse.sql` DDL), `sdk`, `consent-js`,
`host-contracts`, `host-cloudflare-worker`, `worker-core`, `strategies`,
`portal-community`, `deploy` — all under `packages/`, npm scope `@attestrack/*`.
`consent-js`, `worker-core`, `host-cloudflare-worker`, `portal-community` are
`private: true` today.

## Rules

- `@typescript-eslint/no-explicit-any` is an error — never use `any`
- Prevailing style: single quotes, no semicolons (not lint-enforced; match surrounding code)
- Types flow downward (`types` → `schema` → `sdk` → the rest); no circular deps
- Portal: business logic in hooks, components presentational
- **Boundary (ADR-009/010):** never reference the licensed sibling repo outside
  the allowlisted boundary docs (`scripts/boundary-check.mjs` is a CI gate);
  `packages/merkle/` must never exist
- Community consent is OSS scope (ADR-010): `ConsentConfig` KV, mandatory
  strategies (jurisdiction / consent / consent-log), HMAC tokens, `consent-js`
  banner served at `GET /consent.js`. Banner-config / policy-publish / mode
  mutations return `403 requires_attestrue`
- Consent events are KV-only with operator-configurable TTL — no D1/R2 in v1 (ADR-011)
- Worker sets the consent cookie via `Set-Cookie` with `Domain` from site config (ADR-012)
- E2E is Playwright against a Node dev server wrapping the Worker handler with a
  mock host — not Miniflare/workerd (do not claim otherwise in docs)
- Keep `docs/REPO-SPEC-OSS.md`, `docs/oss-http-contract.json`, and contract
  tests in sync when touching Worker routes or error codes

---
> Source: [matt-cochran/attestrack](https://github.com/matt-cochran/attestrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

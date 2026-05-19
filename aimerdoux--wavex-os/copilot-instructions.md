## wavex-os

> Codebase context + AI-agent rules of engagement. **Read this before any task on this repo.**

# CLAUDE.md

Codebase context + AI-agent rules of engagement. **Read this before any task on this repo.**

> **Behavioral guardrails:** see [`~/.claude/CLAUDE.md` → Karpathy Behavioral Guidelines](~/.claude/CLAUDE.md).

## What this repo is

WaveX OS — open-source operating system for running an AI agent company on
localhost. The full-fidelity onboarding pipeline is owned by wavex-os
(vendored at `vendor/wavex-os/`) and surfaced through wavex-os adapter
packages. Wavex-os contributes the dashboard (`MissionControl` +
`components/mission/*`), the runtime layer (healing, observability, launchd
templates, agent skills, install scripts), and the boundary services
(auth, composio, db, inference) that the vendored plugin consumes.

They meet at `~/.wavex-os/instances/<companyId>/`.

## Repo map

```
wavex-os/
├── apps/installer/                  npx wavex-os init CLI
├── packages/
│   ├── core/                        Paperclip vendored via subtree
│   ├── db/                ★ NEW     PGlite (dev) / Postgres (prod) + Drizzle schema
│   ├── plugin-sdk-shim/   ★ NEW     Re-exports @paperclipai/plugin-sdk surface
│   ├── auth-shim/         ★ NEW     assertBoard / assertCompanyAccess (WAVEX_AUTH_MODE)
│   ├── composio-shim/     ★ NEW     listConnections + featured toolkits (WAVEX_COMPOSIO_DISABLED)
│   ├── inference-adapter/ ★ NEW     tier-router claudeBin (WAVEX_INFERENCE_MODE: oauth/apikey)
│   ├── wavex-os-server/   ★ NEW     Fastify routes for vendored onboarding
│   ├── onboarding-ui/               Browser app (MissionControl + WavexOsOnboarding)
│   ├── mock-core/                   Fastify server: hosts /api/* + wavex-os routes
│   ├── healing/             ★ FROZEN    OAuth refresh, worker restart, 401 fallback
│   ├── observability/       ★ FROZEN    bottlenecks, attribution, token budget (drizzle-orm peer)
│   ├── standard-skills/     ★ FROZEN    cross-cutting agent skills
│   └── agent-templates/                  per-role skill definitions
├── vendor/wavex-os/         ★ NEW     vendored wavex-os @ d84983a1 (2026-05-03)
│   ├── plugin-sdk/                       @paperclipai/plugin-sdk
│   ├── shared/                           @paperclipai/shared
│   ├── tier-router/                      @wavex-os/plugin-tier-router
│   ├── flywheel-kernel/                  @wavex-os/plugin-flywheel-kernel
│   ├── onboarding/                       @wavex-os/plugin-onboarding (50 src files)
│   ├── wavex-os-flow-types/              @wavex-os/plugin-flow-types
│   ├── ui-onboarding-components/         upstream UI source (future migration target)
│   ├── tsconfig.base.json                shared sibling base for vendored packages
│   └── VENDOR.md                         source SHA + vendor exceptions + update procedure
├── scripts/wrappers/        ★ FROZEN    claude-anthropic-direct.sh, claude-spawn.sh
├── scripts/wavex-claude-spawn.sh   ★ NEW   T2 spawn shim (prepends `exec` for tier-router)
├── scripts/render-launchd-templates.mjs ★ FROZEN
├── scripts/provision-*.mjs               ★ FROZEN
├── templates/launchd/       ★ FROZEN    macOS plist templates
├── examples/*.example.json  ★ FROZEN    runtime contract spec
├── baseline/                ★ NEW     captured fixture KPI baselines
├── docs/ops/                ★ NEW     surface-tuning-map.md / -chart.html (40 tunables)
├── docs/onboarding/migration-plan.md     full-fidelity port plan + 7 committed decisions
├── docs/MINIMAL_INCEPTION.md             kernel topology spec
└── docs/SELF_HEALING.md                  4-layer recovery architecture
```

## Frozen paths (DO NOT MODIFY)

```
packages/healing/**
packages/observability/src/**            (package.json may add type deps only)
packages/standard-skills/**
packages/onboarding-ui/public/agent-templates/**
apps/installer/**
scripts/wrappers/*.sh
scripts/render-launchd-templates.mjs
scripts/provision-*.mjs
scripts/setup-hierarchy-and-kpis.sample.mjs
templates/launchd/**
examples/*.example.json
vendor/wavex-os/**                       (excepting documented patches in VENDOR.md)
```

If a frozen path needs to change to complete a task, **STOP** and surface the concern. Do not proceed.

## Where to start by task

| Task | Touch |
|---|---|
| Add a pillar field | `vendor/wavex-os/onboarding/src/schema/pillar-responses.ts` upstream + re-vendor |
| Add a connector | `vendor/wavex-os/onboarding/src/phases/phase-2-connector/decision-matrix.ts` upstream |
| Add a Fastify route | `packages/wavex-os-server/src/routes/*.ts` |
| Add an inference call | Plugin's tier-router; wavex-os adapter sets bin via inference-adapter |
| Add a wizard subview | `packages/onboarding-ui/src/wavex-os/{pillars,phases}/` (wavex layout) |
| Add a dashboard read | `packages/wavex-os-server/src/routes/instance.ts` (new endpoint) |
| Add a DB table | `packages/db/src/schema/<file>.ts` + new migration in `packages/db/migrations/` |
| Activate a finalized manifest into the DB | `POST /api/instance/<id>/activate` → `packages/wavex-os-server/src/routes/activate.ts` → `bridge/finalize-bridge.ts` |
| Add a slot↔template mapping | `packages/wavex-os-server/src/bridge/catalog.ts` (`SLOT_TO_TEMPLATE`) |

## Onboarding contract (filesystem)

```
~/.wavex-os/instances/<companyId>/         ← wavex projection layer
└── (kpi-registry derived from /api/instance/<id>/kpis on demand)

~/.wavex-os/instances/default/companies/<companyId>/onboarding/
├── pillar_responses.json                  vendored plugin (mutable draft)
├── connector_manifest.{yaml,json}         vendored plugin
├── swarm_manifest.{yaml,json}             vendored plugin
├── workflow_manifest.{yaml,json}          vendored plugin
├── company.manifest.{yaml,json}           vendored plugin (signed)
├── manifest.sig                           vendored plugin
└── mc-report.json                         vendored plugin (Monte Carlo)
```

## Environment variables

| Var | Default (dev) | Production | Purpose |
|---|---|---|---|
| `WAVEX_AUTH_MODE` | `dev` | `production` | Auth gate behavior (dev bypass / Better-Auth) |
| `WAVEX_COMPOSIO_DISABLED` | `1` (auto if `NODE_ENV!=production`) | unset | Composio integration disable |
| `WAVEX_INFERENCE_MODE` | `oauth` | `apikey` | Tier-router claudeBin source |
| `WAVEX_DB_DRIVER` | `pglite` | `pg` | Drizzle backend |
| `WAVEX_DB_DATA_DIR` | `~/.wavex-os/db/pglite` | n/a | PGlite data dir |
| `DATABASE_URL` | n/a | required when `WAVEX_DB_DRIVER=pg` | Postgres connection string |
| `WAVEX_OS_STATE_DIR` | `~/.wavex-os` | per-deploy | Wavex root |
| `PAPERCLIP_DATA_DIR` | `$WAVEX_OS_STATE_DIR` | per-deploy | Plugin session root (auto-bridged) |
| `WAVEX_OS_CLAUDE_BIN` | (auto) | (auto) | Mutated by inference-adapter at boot |
| `COMPOSIO_API_KEY` | unset | required for live Composio | Composio auth |
| `ANTHROPIC_API_KEY` | unset (dev uses keychain) | required in apikey mode | Claude API |

## Test commands

```bash
pnpm test                                  # @wavex-os/db smoke
pnpm --filter @wavex-os/* test             # all wavex packages (42 tests)
pnpm --filter @wavex-os/* test             # vendored plugin tests (148 tests)
pnpm dev                                    # boots ui (5173) + mock-core (3101)
pnpm db:up                                  # idempotent: creates PGlite + applies migrations
pnpm tuning:map                             # regenerates docs/ops/surface-tuning-map.md
pnpm baseline:capture                       # runs 4 fixtures + writes baseline JSON
pnpm diffeq:suite                           # runs differential-equation harness
```

## Production swap path

When deploying to a real environment:

1. Set `NODE_ENV=production`. Defaults flip to: `WAVEX_AUTH_MODE=production`,
   `WAVEX_INFERENCE_MODE=apikey`, Composio enabled.
2. Provide `DATABASE_URL` + `WAVEX_DB_DRIVER=pg` for real Postgres.
3. Provide `ANTHROPIC_API_KEY` for tier-router T2 calls.
4. Provide `COMPOSIO_API_KEY` for live OAuth orchestration (if used).
5. Wire Better-Auth setup so `req.actor` is populated by middleware before
   the assertion gates run.
6. launchd templates (or systemd equivalents on Linux) drive the healing +
   observability loops against the same `@wavex-os/db` instance.

The vendored wavex-os plugin code (`vendor/wavex-os/`) is byte-identical
to upstream and does not change between dev and production.

## Coding standards

- TypeScript strict mode (wavex-os-server relaxes `noImplicitAny` only for
  Fastify route handler inference quirks)
- Conventional commits: `feat(<area>): ...`, `fix(<area>): ...`, `test: ...`
- One logical change per commit
- No new runtime dependencies in the wavex layer without justification
- Vault: never log plaintext; never persist plaintext to disk

## Related docs

- [`docs/onboarding/migration-plan.md`](./docs/onboarding/migration-plan.md) — full-fidelity port plan + 7 decisions
- [`vendor/wavex-os/VENDOR.md`](./vendor/wavex-os/VENDOR.md) — source SHA + vendor exceptions
- [`docs/ops/surface-tuning-map.md`](./docs/ops/surface-tuning-map.md) — 40 tunables (auto-generated)
- [`docs/MINIMAL_INCEPTION.md`](./docs/MINIMAL_INCEPTION.md) — kernel topology
- [`docs/SELF_HEALING.md`](./docs/SELF_HEALING.md) — runtime recovery

---
> Source: [aimerdoux/wavex-os](https://github.com/aimerdoux/wavex-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

## afi-reactor

> **afi-reactor** is AFI Protocol's **scored-signal evaluation runtime (District One)**.

# afi-reactor — Agent Instructions

**afi-reactor** is AFI Protocol's **scored-signal evaluation runtime (District One)**.
It ingests signals, runs **one** manifest-driven graph executor over five
provider-backed enrichment lanes, produces a UWR score via the Froggy analyst,
and hands the scored signal to District Two evidence/provenance. It **stops at
scored** — validator certification and trade execution are downstream / external.

**Naming Note**: Use "afi-reactor" naming throughout.

**Global Authority**: All agents/droids operating in AFI Protocol repos must
follow `afi-config/codex/governance/droids/AFI_DROID_CHARTER.v0.1.md` (LIVE
authority). If this AGENTS.md conflicts with the Charter, **the Charter wins**.

For global droid behavior and terminology, see:
- `afi-config/codex/governance/droids/AFI_DROID_CHARTER.v0.1.md`
- `afi-config/codex/governance/droids/AFI_DROID_PLAYBOOK.v0.1.md`
- `afi-config/codex/governance/droids/AFI_DROID_GLOSSARY.md`

---

## Architecture (current)

Pipeline composition is **data, not code**: there is no hardcoded pipeline in
source. Flow:

1. **HTTP ingress** — Universal Signal Schema v1.1 via the Gateway signals path
   (reactor endpoint `POST /api/webhooks/tradingview`; the governed
   structured-ingress front is afi-gateway `POST /api/v1/signals`), plus the
   internal CPJ adapter (`POST /api/ingest/cpj`).
2. **AJV validation** — `src/uss` (USS) and `src/cpj` (CPJ) validators.
3. **One manifest-driven GraphExecutor** — `src/pipeline/executor.ts`,
   constructed once in `src/config/runtimeComposition.ts`, running the
   registered graph **`froggy-trend-pullback` v1.1.0**.
4. **Five vendor-neutral, provider-backed enrichment lanes** — technical,
   pattern, sentiment, news, aiMl. Each lane selects an explicit
   **ProviderInstance** from the governed afi-config registries
   (`registries/{providers,provider-instances,provider-bindings,pipelines,analysis-plugins,analyst-strategies}`).
   The static adapter registry in **`src/providers/`** (`adapterRegistry.ts` +
   `adapters/`) is the SOLE enrichment execution seam.
5. **Join → Froggy analyst → UWR score** — the five lane results join, then the
   Froggy analyst scores `trend_pullback_v1` from **afi-core**, producing a UWR
   score (Utility / Work-quality / Rarity; axes: structure, execution, risk,
   insight).
6. **District Two evidence/provenance** — the governed record
   `afi.scored-signal-evidence.v3` is persisted through **afi-infra** (the SOLE
   evidence writer) into MongoDB collection `scored_signal_evidence`.

**Output**: a scored-only signal with
`analystScore { uwrScore, uwrAxes { structure, execution, risk, insight } }`
(plus `rawUss`, optional `lenses`, optional `_priceFeedMetadata`, `scoredAt`,
`decayParams`, `meta`). There is **no** `validatorDecision` and **no** execution
block.

**Not the reactor's responsibility** (stops at scored): validator certification
and trade execution are downstream / external; mint orchestration lives in
`afi-mint`. The legacy Froggy demo personas (Alpha Scout / Pixel Rick / Val Dook
/ Execution Sim) were removed and must not return.

Scoring, UWR, decay, and evidence are unchanged and **byte-stable** (oracle
goldens).

---

## Build & Test

```bash
npm install              # install deps (siblings afi-core/afi-math/afi-config linked via file:../)
npm run build            # tsc → dist
npm run typecheck        # tsc --noEmit
npm test                 # jest: unit + guardrails + behavioral oracle goldens
npm run start:demo       # node dist/src/server.js (port 8080)
npm run esm:check        # ESM invariant lint
```

Real-Mongo proofs (gated; CI runs them):

```bash
npm run test:integration:unavailable   # 503 smoke when the store is unavailable
npm run test:integration:mongo         # evidence persistence
npm run test:integration:shutdown      # graceful shutdown
npm run test:oracle:mongo              # oracle byte-equivalence against real Mongo
```

Never regenerate oracle goldens to make a change pass — byte drift is the signal
that behavior changed.

---

## Graph Executor

The single execution path is the generic graph executor; composition is loaded
and validated at boot:

- `src/pipeline/registryLoader.ts` — loads + AJV-validates the governed
  registries (pipelines, analysis plugins, analyst strategies, provider
  bindings/instances) and verifies canonical `afi.hash.v1` pins at boot
  (fail-closed; the server refuses to boot on an invalid registry).
- `src/config/runtimeComposition.ts` — the boot-validated composition seam
  (`initRuntimeComposition`); tests may inject overlay registry roots.
- `src/config/strategyResolution.ts` — resolves the provider binding to a
  registered strategy triple (never free text).
- `src/pipeline/executor.ts` — executes the REGISTERED pipeline manifest
  (categories, ports, join determinism, failure policies) over the plugin
  registry (`src/pipeline/pluginRegistry.ts`).
- `src/providers/` — the provider-adapter framework; `adapterRegistry.ts` +
  `adapters/` is the SOLE enrichment execution seam (secrets only via the
  injected `SecretResolver`; never a key in a URL; output validated against the
  governed category contract).
- `src/evidence/` — evidence construction + District-2 provenance law
  (`provenance/`: CanonicalHash v1, projection builders, D2 schema validation;
  `analysis/`: the internal scoring carrier).

Guardrail: `test/guardrails/no-hardcoded-composition.test.ts` — there is no
hardcoded pipeline in production source.

---

## Repo Layout

```
src/     # runtime source: pipeline/ providers/ config/ evidence/ uss/ cpj/
         # services/ adapters/ collectors/ core/ enrichment/ indicator/ news/
         # utils/ types/  + server.ts
test/    # jest unit + guardrails + oracle goldens + integration-mongo proofs
docs/    # architecture + operational docs
droids/  # droid orientation (00_repo_orientation, 10_common_tasks, 20_safe_patch_patterns)
scripts/ # esm-check.sh and helpers
```
Root config: `package.json`, `tsconfig.json`, `jest.config.js`, `typings.d.ts`.

**Depends on**: afi-core (scoring, ESM package via `file:../`), afi-config
(governed registries + canonical USS schema), afi-infra (canonical evidence
store — sole writer).
**Consumed by**: gateways (e.g. afi-gateway) as external HTTP clients.

---

## ESM Invariants

afi-reactor is **pure ESM**. Required practices:

- Import from **afi-core** by package name, never cross-repo relative paths:
  ```typescript
  // ✅ CORRECT
  import { scoreFroggyTrendPullbackFromEnriched } from "afi-core/analysts/froggy.trend_pullback_v1.js";
  // ❌ WRONG — cross-repo relative path breaks at runtime
  import { scoreFroggyTrendPullbackFromEnriched } from "../../afi-core/analysts/froggy.trend_pullback_v1.js";
  ```
- All relative imports within afi-reactor **must** include `.js` extensions
  (`tsc`-only build, no bundler; Node ESM requires explicit extensions).
- No imports may reference `.ts` files at runtime. No CommonJS.

Validate with `npm run build` and `npm run esm:check`.

---

## Security & Boundaries

- Secrets only through the injected `SecretResolver`; never a key in a URL,
  never a secret in a registry or manifest.
- afi-reactor MUST NOT import ElizaOS code, SDKs, or character definitions.
  Gateways depend on afi-reactor via HTTP; afi-reactor never depends on them.
- `POST /api/ingest/cpj` is an **internal trusted service boundary** — not a
  public API and not a route around the Gateway. See
  `docs/HTTP_WEBHOOK_SERVER.md`.
- Evidence is written only by afi-infra. The reactor builds/validates
  `afi.scored-signal-evidence.v3` and submits it — it is not the store owner.

---

## Git Workflow

- **Base branch**: `main`. Direct pushes to `main` are disabled; all changes
  land via PRs from short-lived `feature/`/`fix/`/`mission/` branches derived
  from `origin/main`.
- **Before committing**: `npm run build && npm test`.
- See `docs/BRANCH_DOCTRINE.v0.1.md` for the full branch rules.

---

## Scope & Boundaries for Agents

**Allowed**:
- Add provider adapters under `src/providers/adapters/` (registered once via
  `builtinProviderAdapters()` in `src/providers/index.ts`).
- Add pipeline nodes under `src/pipeline/nodes/` and register them in
  `src/pipeline/pluginRegistry.ts` with a pinned `pluginId@version`.
- Add tests; extend the governed registries (keeping hash discipline intact).

**Forbidden**:
- Change the registered composition model / manifests without explicit approval.
- Regenerate oracle goldens to force a change to pass.
- Add validator-certification, execution, or evidence-writing logic that belongs
  downstream / to afi-infra.
- Import ElizaOS or make the reactor depend on a gateway.

**When unsure**: prefer a no-op and ask for an explicit spec before changing the
composition model, scoring, or the evidence boundary.

---

## Dev Environment Notes

Non-obvious notes for running afi-reactor in a fresh checkout / cloud VM.
Standard commands are in **Build & Test** above; only caveats are here.

### Sibling repos are mandatory and live beside the repo

`package.json` links `afi-core` and `afi-config` via `file:../` (afi-core also
pulls in `afi-math`). With the repo at `/workspace`, `../` resolves to `/`, so
the siblings must exist at `/afi-core`, `/afi-math`, and **`/afi-config`**
(required since afi-config became a real schema dependency of the canonical USS
validator). afi-factory is NOT a dependency: authoring stays in afi-factory, and
the executor conformance fixtures are vendored byte-copies under
`test/pipeline/fixtures/conformance/`. If a sibling is missing, clone it beside
the repo:

```bash
for r in afi-core afi-math afi-config; do
  sudo mkdir -p /$r && sudo chown -R "$USER" /$r
  git clone https://github.com/AFI-Protocol/$r.git /$r
done
```

### afi-config's prepare/build can affect installs

`afi-config` declares a `prepare` script (`npm run build` → `tsc`). Because
afi-reactor depends on `afi-config@file:../afi-config`, a plain `npm install`
in afi-reactor can trigger that hook — which may fail on a bare afi-config
checkout and will regenerate `afi-config/dist/**` (harmless drift; restore with
`git -C /afi-config restore dist`). Mitigations: run `npm ci` inside
`/afi-config` first (CI already builds afi-config before installing
afi-reactor), or install with `npm install --ignore-scripts` when you only need
dependency resolution.

### Runtime deps required for server bootability

`npm run start:demo` (`dist/src/server.js`, port 8080) statically imports
`ccxt` (BloFin/Coinbase price-feed adapters) and `telegram` /
`node-telegram-bot-api` / `input` (Telegram collectors — opt-in at runtime but
statically imported). These are declared in `package.json`; without them the
server exits at startup with `ERR_MODULE_NOT_FOUND`.

### Running / smoke-testing the server

`npm run build` then `npm run start:demo`. Healthy startup logs
`USS v1.1 validator initialized successfully` and
`Listening on http://localhost:8080`. The `❌ Missing required MTProto env vars`
line is expected — Telegram collectors are opt-in
(`AFI_TELEGRAM_MTPROTO_ENABLED=1`). No env vars or MongoDB are required for the
demo flow.

```bash
curl -s localhost:8080/health
curl -s -X POST localhost:8080/api/webhooks/tradingview -H 'Content-Type: application/json' \
  -d '{"symbol":"BTC/USDT","timeframe":"1h","strategy":"trend_pullback_v1","direction":"long"}'
```

The webhook returns a scored-only signal (`analystScore.uwrScore` + `uwrAxes`;
no validator/execution block). Note: the repo-wide `npm run esm:check` still
flags a handful of pre-existing files importing without `.js` extensions (e.g.
`test/uss/*`); this is long-standing and does not affect build/test.

---

**Maintainers**: AFI Reactor Team | **Charter**: `afi-config/codex/governance/droids/AFI_DROID_CHARTER.v0.1.md`

---
> Source: [AFI-Protocol/afi-reactor](https://github.com/AFI-Protocol/afi-reactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

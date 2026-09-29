## livepeer-modules-suite

> > **Map, not manual.** This file is the table of contents for the Livepeer Modules

# AGENTS.md

> **Map, not manual.** This file is the table of contents for the Livepeer Modules
> Suite. It points to the system of record in `docs/`. Keep it short (~100 lines).
> When in doubt, **link — don't inline.** Operating principles live in
> [`docs/design-docs/core-beliefs.md`](docs/design-docs/core-beliefs.md).

## What this repository is

The **Livepeer Modules Suite** is an umbrella (meta) repository. It aggregates the
individual Livepeer Module repositories as **git submodules** and provides a single,
high-level, agent-legible overview of the whole suite and how its parts fit together.

- This repo holds **documentation and submodule pointers** — not module source code.
  Each module's code lives in its own repository, mounted under `modules/<name>/`.
- Goal: an agent or human can understand the entire suite and the relationships
  between modules **directly from this repository**.
- The Livepeer Modules are the productized features that enable the Livepeer
  protocol. They are developed independently and released on the network's cadence;
  this suite tracks specific submodule revisions so the overview always corresponds
  to a known, reproducible set of releases.

## Two axes: capabilities and repos

- **Capabilities** (the "what") — the conceptual feature areas, one spec each in
  [`docs/product-specs/`](docs/product-specs/index.md).
- **Repos** (the "where") — the actual submodules, one doc each in
  [`docs/repos/`](docs/repos/index.md). **A single repo can implement several
  capabilities** (e.g. `livepeer-network-modules` covers most of the supply side).

New terms? Start with the [`docs/glossary.md`](docs/glossary.md).

## The capabilities (the suite)

One line each. Full specs in [`docs/product-specs/`](docs/product-specs/index.md).

| Capability | One-liner | Spec |
| --- | --- | --- |
| Gateways | Demand-side entry point: discover, pay, forward work | [gateways](docs/product-specs/gateways.md) |
| Orchestrators | Supply side: a workload-agnostic broker + daemons that serve paid work | [orchestrators](docs/product-specs/orchestrators.md) |
| Runners | The backends a broker dispatches to (the provided capability) | [runners](docs/product-specs/runners.md) |
| Pools | Control plane aggregating member backends behind one orch identity | [pools](docs/product-specs/pools.md) |
| Payment | Probabilistic micropayment tickets exchanged for work | [payment](docs/product-specs/payment.md) |
| Service Registry | On-chain pointer + off-chain signed manifest of capabilities | [service-registry](docs/product-specs/service-registry.md) |
| Discover | Resolver API gateways use to find and select orchestrators | [discover](docs/product-specs/discover.md) |
| Payment Clearinghouse | Settlement/distribution of payments (+ SDKs) | [payment-clearinghouse](docs/product-specs/payment-clearinghouse.md) |
| SDKs | Client/integration libraries spanning the suite | [sdks](docs/product-specs/sdks.md) |
| Reference Apps | Example gateway apps showing how to build on the suite | [reference-apps](docs/product-specs/reference-apps.md) |

### Observability & Reporting (off-network — tracks on-chain activity)

| Capability | One-liner | Spec |
| --- | --- | --- |
| Protocol Explorer | Indexes + prices + serves Livepeer on-chain activity (API + web explorer) | [protocol-explorer](docs/product-specs/protocol-explorer.md) |
| Network Bot | Reports payouts/activity to Discord by polling the explorer | [network-bot](docs/product-specs/network-bot.md) |

## Repositories (submodules)

| Repo | Implements | Doc |
| --- | --- | --- |
| livepeer-network-modules | Supply-side core: Orchestrators, Runners, Pools, Payment, Service Registry, Discover, Protocol, SDKs | [repos/livepeer-network-modules](docs/repos/livepeer-network-modules.md) |
| livepeer-open-clearinghouse | Demand-side control plane: Payment Clearinghouse, Payment issuance, SDKs, Discover proxy, gateway control-plane half. Consumes network-modules daemons. | [repos/livepeer-open-clearinghouse](docs/repos/livepeer-open-clearinghouse.md) |
| livepeer-modules-openai-runners | Runner backends: OpenAI/Cohere-shaped AI services (chat, embeddings, audio, TTS, image, rerank) that sit behind the capability broker. | [repos/livepeer-modules-openai-runners](docs/repos/livepeer-modules-openai-runners.md) |
| livepeer-modules-transcode-runners | Runner backends: video (VOD transcode, ABR ladder, live RTMP→HLS). live-runner = `live-session-gateway-ingest@v0` ("Option B"). | [repos/livepeer-modules-transcode-runners](docs/repos/livepeer-modules-transcode-runners.md) |
| livepeer-modules-transcode-gateway | Full in-path video Gateway (VOD ABR + live RTMP→HLS). Operator-funded via LOC; no local payer/resolver daemons. | [repos/livepeer-modules-transcode-gateway](docs/repos/livepeer-modules-transcode-gateway.md) |
| livepeer-modules-openai-gateway | Full in-path OpenAI-compatible AI Gateway ("change `base_url`"). TS/Fastify LOC-mediated twin of transcode-gateway; fronts the openai-runners. | [repos/livepeer-modules-openai-gateway](docs/repos/livepeer-modules-openai-gateway.md) |
| livepeer-protocol-explorer | **Observability.** Rust data platform: indexes/prices/serves Livepeer on-chain activity (API + SPA). Reads chain directly. | [repos/livepeer-protocol-explorer](docs/repos/livepeer-protocol-explorer.md) |
| livepeer-network-bot | **Observability.** Discord bot reporting payouts/activity; polls the protocol-explorer API. | [repos/livepeer-network-bot](docs/repos/livepeer-network-bot.md) |

The end-to-end picture (how a unit of work flows through these capabilities) is in
[`ARCHITECTURE.md`](ARCHITECTURE.md).

## Repository map

```text
AGENTS.md            <- you are here: the map
ARCHITECTURE.md      <- top-level domain map + job-flow across modules
modules/             <- git submodules, one per repo (see modules/README.md)
docs/
├── glossary.md      <- shared vocabulary — start here for unfamiliar terms
├── design-docs/     <- operating principles + design index (core-beliefs.md, index.md)
├── product-specs/   <- capability axis: one overview per capability
├── repos/           <- repo axis: one doc per submodule (index.md + <repo>.md)
├── guides/          <- how-tos for maintaining this repo (git-submodules-primer.md)
├── references/      <- external reference material (e.g. the harness-engineering post)
├── exec-plans/      <- active/, completed/, tech-debt-tracker.md
└── generated/       <- machine-generated docs (e.g. submodule manifest)
```

## Working in this repository

- **Onboarding a repo (user hands you one):** add it under `modules/<name>/`, read it,
  then (1) write/refresh `docs/repos/<name>.md` with a component map + pinned revision,
  (2) update the capability specs in `docs/product-specs/` it implements — replacing
  hedged language with confirmed facts and **resolving open questions as the code
  answers them**, (3) add new terms to [`docs/glossary.md`](docs/glossary.md), and
  (4) update the index tables here and in `docs/repos/`. Document only what the repo
  actually shows.
- **Submodules (clone, update, pin to releases):** see
  [`docs/guides/git-submodules-primer.md`](docs/guides/git-submodules-primer.md).
- **Planning larger work:** capture it in [`docs/exec-plans/`](docs/exec-plans/);
  track shortcuts in [`tech-debt-tracker.md`](docs/exec-plans/tech-debt-tracker.md).

## Conventions

- **Progressive disclosure:** start here, follow links. Don't grow this file into an
  encyclopedia — push detail into `docs/`.
- **Agent legibility:** if it isn't in this repo as versioned markdown (or a tracked
  submodule), it effectively doesn't exist. Capture decisions here, not in chat.
- **Honesty over completeness:** mark unverified module claims as drafts. A correct
  "we don't know yet" beats a confident guess.
- **Keep the map current:** when a module's status changes, update both its spec and
  the table above.

## Status

Eight repos onboarded across two areas. **Network:** `livepeer-network-modules`
(supply-side core), `livepeer-open-clearinghouse` (Payment Clearinghouse),
`livepeer-modules-openai-runners` + `livepeer-modules-transcode-runners` (AI + video Runner
backends), and two full LOC-mediated gateways — `livepeer-modules-openai-gateway` (AI)
and `livepeer-modules-transcode-gateway` (video); LOC is now the shared route/payment
control plane for those gateways while the customer SDK handoff path remains a distinct
demand-side shape. **Observability
(off-network):** `livepeer-protocol-explorer` (on-chain data platform) and
`livepeer-network-bot` (Discord reporting; polls the explorer). Most capabilities are now
🟠 Documented; a daydream gateway, vtuber runners, and Reference Apps await their own repos.
Status: [`docs/product-specs/index.md`](docs/product-specs/index.md) (capabilities),
[`docs/repos/index.md`](docs/repos/index.md) (repos).

---
> Source: [Cloud-SPE/livepeer-modules-suite](https://github.com/Cloud-SPE/livepeer-modules-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->

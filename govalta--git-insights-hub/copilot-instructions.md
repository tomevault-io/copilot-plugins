## git-insights-hub

> Organization-wide GitHub analytics + a multi-detector security scanning pipeline. This file

# Git Insights Hub — repository guide

Organization-wide GitHub analytics + a multi-detector security scanning pipeline. This file
orients contributors (and Claude Code sessions) working in this repo. User-facing docs:
[README.md](README.md), [SCANS.md](SCANS.md), [docs/](docs/).

## Stack & conventions
- **Node 22, ESM** (`"type": "module"`). Express API + Vue 3 (CDN) dashboards in `public/`.
- **PostgreSQL 14+**; schema is created/updated by idempotent migrations in `db/migrate.js`
  (`npm run migrate`). Migrations are append-only — add a new numbered entry, never edit an
  applied one.
- **LLM provider** is pluggable (`lib/llm-provider.js`, `LLM_PROVIDER`): **Anthropic direct**
  (`anthropic`, standard commercial terms - for **unclassified** workloads) or **Claude on Google
  Vertex AI** (`vertex`, Enterprise Agent Platform, stays in the GCP enterprise tenancy under the
  enterprise agreement - for **Protected A/B** workloads). This classification mapping is the
  Government of Alberta's under its current STRA; it is deployer configuration, not a code property
  (see README "Choosing an LLM provider"). The two **opt-in** Agent-SDK paths
  (`AGENT_VERIFIER=1`, `bcm-scan --max`/`BCM_DEEPDIVE_AGENT=1`) use `@anthropic-ai/claude-agent-sdk`,
  which talks to Anthropic **direct** regardless of `LLM_PROVIDER`; both default off, so a Vertex
  Protected deployment stays on Vertex unless they are explicitly enabled.
  Tiers: `fast` = Sonnet (`claude-sonnet-5`), `deep` = Opus (`claude-opus-4-8`). Pick a tier
  with `getProviderForTier("fast"|"deep")`. **`temperature` is DEPRECATED on the flagship model**
  (Opus 4.8 on Vertex returns `400 "temperature is deprecated for this model"`), so it is NOT a
  usable determinism lever — the provider sends it only when explicitly opted in and drops it on
  the deprecation error. Run-to-run consistency of LLM steps must therefore come from *constraining
  the task* (compute numbers in code, tighten schemas), not from settings.
- **Decision-grade numbers are computed in code, never by the LLM.** The rebuild estimate lives in
  `lib/reporting/estimate.js` (`computeEstimate`): deterministic unit counts × per-unit rates parsed
  from `profiles/<org>.estimation.md` + a signal-derived complexity mix. Identical inputs → identical
  estimate (was ~34% drift when the model did the arithmetic in-prompt). `chief-architect.js` calls
  it and discards the model's `estimate`; the model only writes narrative. Tests: `tests/estimate.test.js`.
- **Reproducibility invariants (do not regress).** Because temperature
  can't pin the model, run-to-run consistency is engineered structurally: (1) every filesystem walk
  that feeds an LLM prompt or a persisted array is **sorted** (`file-inventory.js`, `import-graph.js`,
  `robust-scan.js` walks) and every `SELECT` feeding a prompt/dedup has an `ORDER BY`; (2) no LLM
  owns a **decision-grade number** — severity is derived in `consensus.js` from source detectors +
  a boolean verifier gate (not the verifier's tier enum), scores/estimate/disposition are computed
  in code, BCM primitive counts are **deduped** before counting, and all business-case option costs
  are code-locked; (3) LLM calls are **truncation-safe** — providers signal `{_truncated, partial}`
  and callers split/salvage instead of silently dropping (`code-scanner.js` splits batches); (4) the
  capability taxonomy is **order-invariant** (lexicographically-smallest canonical wins in
  `store.js`). Model-drawn Mermaid diagrams remain narrative (labeled as illustrative). Every LLM
  call retries all transient failures (5xx / network / rate-limit / empty-response) with backoff
  (`LLM_MAX_RETRIES`, default 5). For the residual set-membership drift in the two enumerated LLM
  stages, **multi-sample consensus** is opt-in via `CONSENSUS_SAMPLES` (N, default 1) + `CONSENSUS_MIN`
  (K, default majority): the AI generator (`code-scanner.js`) and BCM deep-dive (`deep-dive.js`) each
  sample N times and keep items recurring in >= K (pure helper `lib/util/consensus.js`).
- **Nothing org-specific is hardcoded** — it lives in an org profile (`profiles/*.json`,
  selected by `ORG_PROFILE`; `default.json` is the empty generic base, `example.json` is a complete
  generic worked example used by the test suite). Profile values are SQL-escaped via `lib/profile-sql.js`.
  Org-specific data that used to be hardcoded in shared code now lives in profile keys: repo→domain
  buckets (`dashboard_domain_patterns`, consumed by `dashboardDomainCase` in the dashboard SQL + the
  client drill-down via the capability-map API), content→domain keywords (`content_domain_keywords`,
  `capability-scanner.js`), external-system canonicalization (`external_system_aliases`,
  `architecture-map.js`), and repo family prefixes (`repo_family_prefixes`). All ship empty in
  `default.json`; an org fills in its own. Tests run under `ORG_PROFILE=example` (`tests/test-profile`),
  so the suite never depends on any real org's profile.
- **Single generic version / deployment & anonymity.** The tracked tree is 100% generic — no real
  system/repo names anywhere in committed code or config, so ONE codebase serves every org and any
  number of remotes (e.g. a public mirror and a private/internal remote) carry
  **byte-identical** tracked trees (a `git diff` of tracked files between them is empty). The real
  operational profile (`profiles/alberta.json` — real org, prefixes, program-code maps) is **gitignored**:
  it exists only on the operating machine, never committed to either remote. Adopters copy `example.json`
  (or `default.json`) to their own `profiles/<org>.json` and set `ORG_PROFILE`. Publish to both remotes
  with the same commit; there is no scrub step and nothing to diverge.
- **Secrets** live only in `.env` (gitignored). Never commit `.env`, `data/`, `clones/`,
  `reports/`, or `*.report.json` — generated reports embed real source code and are confidential.

## The robust security pipeline (the heart of this repo)

Entry point: **`scripts/robust-scan.js`** (one repo) → **`scripts/portfolio-scan.js`** (whole
portfolio, resumable). Stages, in order:

1. **Clone** (or `--repo-dir <path>` to scan a local tree). ⚠️ A plain clone does **not**
   recurse git submodules — see gotchas.
2. **Framework detection + reachability** (`lib/reachability/`): per-framework entry-point
   parsers (express, nextjs, dotnet incl. WCF, django, flask, spring, magento2) + a
   bidirectional **import graph** (`import-graph.js`) used to build entry-point-rooted
   **scan clusters** (`lib/security/scan-clustering.js`).
3. **Detectors, in parallel:**
   - **Semgrep** (`lib/security/semgrep-runner.js`) — language + cloud rule packs. Resilient:
     if a rule pack errors, it retries once with the baseline pack so one bad pack can't zero
     out the whole layer.
   - **Secrets** (`lib/security/secret-scanner.js`) — regex + entropy, zero deps.
   - **Dependencies** (`lib/security/dependency-scanner.js`) — OSV.dev query + CVSS enrichment.
   - **AI generator** (`lib/code-scanner.js`) — Sonnet reads the clustered key files.
4. **Merge / dedup** (`lib/security/consensus.js`): dep findings dedup by CVE id, code findings
   by file+line±10 + CWE.
5. **AI verifier** (`lib/security/llm-verifier.js`, one-shot Opus; or `agent-verifier.js` with
   Read/Grep tools when `AGENT_VERIFIER=1`) re-checks every candidate above info tier.
6. **Consensus scoring** → final severity per finding.
7. **Persist + deterministic health scores** (`lib/security/health-score.js`,
   `finding-scoring.js`): scores are computed from confirmed findings + reachability, **not**
   model opinion. Coverage is split into `tree_cluster_coverage_pct` (deep AI) and
   `total_scan_coverage_pct` (any detector).
8. **Scan integrity & coverage gate** (`lib/security/integrity-gate.js`) — a one-pass
   **self-audit** (see below).
9. **Architecture diagram** (`lib/reporting/architecture-diagram.js`) — an LLM pass infers a
   Mermaid model (10–50 nodes) of internal components + external dependencies (APIs, DBs, cloud,
   queues, identity) from frameworks, entry points, dependency manifests, and grepped integration
   tells. Pre-rendered to inline SVG (`lib/reporting/mermaid-render.js`, using the PDF Chrome
   binary + vendored `lib/reporting/vendor/mermaid.min.js`) and shown in the report. Persisted to
   `scan_metrics.architecture_*` (migration 24). Disable with `--skip-architecture`.

### How much code the AI reads (depth/coverage)

Two selectors exist. Coverage % comes from `buildScanClusters` (entry-point → import-graph
neighbors, leftovers capped at `CLUSTER_LEFTOVER_MULTIPLIER`× the budget). What the AI generator
**actually reads** is chosen by `selectAiScanFiles` in `robust-scan.js`: reachable code first (the
entry-rooted clusters, nearest-to-entry first), then leftovers, then a union of high-priority
keyword files (auth/crypto/config); falls back to the flat keyword `pickKeyFiles` when there is no
import graph. Caps (`SCAN_MAX_KEY_FILES`=600, `SCAN_MAX_TOTAL_MB`=8, per-file `SCAN_MAX_FILE_KB`)
are env-tunable and must stay consistent between `robust-scan.js` and `code-scanner.js` (both read
them). A repo under the caps is read in full. NOTE: there is **not yet** recursive post-verification
follow-up (re-scanning the neighbors of confirmed findings) — that requires splitting
`code-scanner.js` into analyze/persist because `scanRepoCode` currently `DELETE`s+reinserts
`code_issues` on every call, so it can't be invoked per-round as-is.

Re-run consensus on existing findings without re-scanning: `scripts/reaggregate.js`.
Reports: `scripts/generate-report.js` → `lib/reporting/` → standalone HTML + PDF in `reports/`.

## Scan integrity & coverage gate — read this before trusting a "clean" result

The detectors don't audit themselves. A scan can complete while silently missing code (crashed
analyzer, unpopulated submodules, unsupported dependency manager, unrecognized framework),
yielding a confident-but-shallow "all clear". Stage 8 exists to catch exactly that:

- **Deterministic layer** (always on, no cost): flags `semgrep-suspicious-clean` (analyzer ran
  but found 0 on a large repo), `submodules-missing` (declared but unpopulated), `deps-unassessable`
  (no manifest → dependency health is UNKNOWN, not clean), and low coverage. Worst check status
  sets confidence: any `fail` → `low`; any `warn` → `medium`; else `high`.
- **One Opus meta-review pass** (skip with `--skip-integrity`): writes the confidence narrative,
  probable errors + concrete remediation, and a `do_not_trust` list.

Output is persisted to `scan_metrics` (`integrity_*` columns, migration 23) and rendered as the
**Scan integrity & confidence** section at the top of every report. Tuning thresholds live in
`THRESHOLDS` in `integrity-gate.js`. Tests: `tests/integrity-gate.test.js`.

**Design principle:** the pipeline is deterministic and reproducible by design; the gate makes
degradation *loud* but does not (yet) auto-correct it — it recommends the fix and a human re-runs.

## Business Capability Map (`scripts/bcm-scan.js`) + coverage inventory

Separate, on-demand, heavier **augmenting** scan — NOT part of every `robust-scan`.
- **Full file inventory** (`lib/scan/file-inventory.js`): walks EVERY file, categorizes it
  (source/config/service-def/manifest/infra/test/doc/data/binary), and runs deterministic security
  "feelers". Used by BOTH robust-scan (coverage denominator + signal-driven file selection, via
  `scan_file_inventory`, migration 25) and bcm-scan. `fileSignalScore()` ranks danger for
  force-inclusion.
- **BCM pipeline** (`lib/bcm/`): `buildFileSets` (directory grouping — every business-relevant
  file lands in one set) → `classifyBatch` (fast-tier, open-minded capability mapping) →
  deep dives (functions/workflows/screens) → `resolveOrCreateCapability` (store.js — canonical,
  organically-growing taxonomy; dedup by slug + alias + token-overlap; seeded from the 74-item gov
  taxonomy but NOT constrained). Tables: `business_capabilities`, `file_capability_map`,
  `v_bcm_capabilities` (migration 26).
- **Two BCM layers:** per-app (a repo's `file_capability_map` rows) and universal/whole-of-gov
  (`business_capabilities` + `v_bcm_capabilities`, grown by every scan). Runnable per repo or org.
- **Render:** inline SVG treemap (`lib/reporting/treemap-render.js`, zero-dep squarified) →
  report section "3c. Business capabilities & functions"; report-data.js loads the latest bcm-scan.

## Unclassified whole-of-gov export ("DataPack") — `npm run datapack`

A distributable, **de-identified** dataset (+ PDF overview + interactive explorer) describing *what
government software does* for vendors/partners, built from a completed fleet run. Entry point:
**`scripts/datapack.js`** (`npm run datapack -- --run <fleetRunId>`), which runs the close-out in the
ONE correct order. Pieces:
- **`scripts/bcm-abstract.js`** — rewrites each raw BCM capability (named after specific apps) into a
  generic, proper-noun-free function label. Seeded from the taxonomy, context-enriched with
  `inferred_ministry` + repo description. Has a **reconcile-retry loop** so a transient LLM batch
  failure never orphans a capability (orphans -> NULL -> excluded from export).
- **`scripts/bcm-consolidate.js`** — merges near-dup labels into a canonical taxonomy.
  **INCREMENTAL + MONOTONIC by default**: the canonical set (`business_capabilities.abstraction_model
  = 'canonical'`) is **FROZEN** and only grows (new labels fold in or are added; frozen ones are never
  re-merged). `--rebuild` re-clusters from scratch (deliberate refresh). Migration 35
  `bcm_taxonomy_baseline` records each baseline. **INVARIANT: adding repos must grow/hold the taxonomy,
  never shrink it** — do not re-run plain consolidate expecting compression.
- **`scripts/bcm-derive-domains.js`** — the tier-1 **domains are DERIVED bottom-up from the actual
  canonical capabilities, NOT from the legacy `SEED_DOMAINS`** (which is a stale prior-AI guess and must
  not be treated as ground truth). The model proposes the domain set the data supports (run-3 estate =>
  ~35), then classifies every capability into it. INCREMENTAL by default (classify new caps into the
  frozen `profiles/<org>.derived-domains.json` vocabulary); `--refresh` re-derives. Domains are thus an
  emergent property of the estate, frozen + reproducible.
- **`scripts/export-gov-metadata.js`** — emits the shared JSONs. **Allowlist-only**; capability text is
  the persisted generic abstraction; `aligned_ministry` is a best-fit **generic portfolio** (real
  ministry only in the withheld crossref); technology **vintage** = real year/age from
  `lib/reporting/tech-vintage.js` (org-editable `profiles/<org>.tech-vintage.md`; tier-B exact versions
  from `scan_metrics.tech_versions` when present); honest bands (`not_scanned` / `unknown` / `none`
  distinguished, never conflated); k-anonymity; org/system-token hard gate; Opus screen. Outputs to
  `exports/` (gitignored): `repos.json`, `capabilities.json`, `tech_stacks.json`, `README.md`,
  `provenance.json` (SHARED) + `AUDIT.json`, `crossref.json` (WITHHELD `_DO_NOT_DISTRIBUTE`).
- **`scripts/gov-datapack-report.js`** — `DATAPACK-OVERVIEW.pdf` (self-contained, inline-SVG charts).
- **`scripts/gov-datapack-viz.js`** — `DATAPACK-EXPLORER.html`: self-contained treemap (domain ->
  capability, recolorable lens: vintage/dep-health/risk/platform), domain heatmap, consolidation +
  modernization tables, and **drill-in** (capability -> app UUIDs -> full released metadata card).

**Reproducibility invariants (do not regress):** (1) taxonomy + domains are FROZEN and only grow across
fleet updates (monotonic); (2) domains come from the data, never re-anchored to SEED_DOMAINS; (3) the
export reads persisted columns only -> byte-deterministic (verify: two exports diff identical);
(4) shared files carry no org/system/app names (org-token hard gate must be 0); (5) empty-default-branch
repos are recovered at clone time (see gotcha) so scans aren't silently empty. Independent leak audit
lives in the export self-check + a manual grep of the shared JSONs before declaring shareable.

## Writing style (house rule)

Reports and LLM analysis use **no em dashes (—) or en dashes (–)** (commas/colons/parentheses instead; hyphen for numeric ranges). Enforced two ways: `STYLE_GUIDE` (`lib/util/style.js`) is appended to LLM prompts, and `finalizeReportStyle()` runs a deterministic pass over the whole assembled report HTML (`report-html.js`) that converts every em/en dash but preserves the lone `—` "no data" table placeholder. `enforceStyle()` also sanitizes stored analysis at generation (Chief Architect, README). Tests: `tests/style.test.js`.

## Ministry / portfolio meta-analysis (`scripts/portfolio-report.js`)

A standalone report across a SET of already-analyzed repos (2-200) that rolls up the persisted per-repo analysis (no rescan) and adds one LLM meta-pass. Focus: opportunity, architecture, costing, capabilities, health, and a meta business case; granular cyber findings stay in the per-repo reports.

- **Data** (`lib/reporting/portfolio-data.js`): `gatherPortfolioData(fullNames)` reuses `gatherReportData()` per repo and rolls up (disposition histogram, summed rebuild cost/days, avg health, cross-repo shared-capability consolidation). `resolveRepoRefs({repos,scanIds})`.
- **Application families** (`--families <json>`): a real system is usually MANY git repos (front end, back end, batch, database, integrations). `gatherFamilyPortfolioData(familyDefs)` groups repos into config-declared families (each `{name|app, prime, supporting[]}`) and attaches a per-family rollup (`pf.families`, rendered as the "Application families" table): cost/days/units/criticals **summed**, risk + timeline the **MAX** single-repo value (a system is only as shippable as its weakest part; the longest critical path bounds the calendar), plus a scanned/total repo count. **Config-driven, no heuristic exclusion** — a family contains exactly the repos it lists; whatever the config omits is simply not in a family (any point-in-time "set these aside" call, e.g. failed-replacement instances, lives in the config, never in code). Pure rollup is `rollupFamilies(repos, familyDefs)`; tested in `tests/portfolio-families.test.js`. The flat `--repos`/`--scan-ids` path is unchanged.
- **Meta pass** (`lib/reporting/portfolio-analysis.js`): deep-tier; emits exec summary, system interplay, consolidated capabilities, health, rolled-up costing (locked to the summed estimates), a suite-level meta business case, a future-state vision + target-architecture Mermaid, and a modernization roadmap. Cached in `portfolio_analyses` (migration 30) by group_key (repo-set hash); recompute with `--refresh`.
- **Render** (`lib/reporting/portfolio-report-html.js`): standalone HTML/PDF reusing the per-repo report styling + em-dash enforcement.
- **CLI:** `npm run portfolio-report -- --repos owner/a,owner/b --title "My System"` (or `--scan-ids`, `--families <json>`, `--out`, `--refresh`, `--no-pdf`). Env: `PORTFOLIO_MODEL_TIER`, `PORTFOLIO_OUTPUT_TOKENS`.

## Gotchas (hard-won)
- **The Agent SDK (`@anthropic-ai/claude-agent-sdk`) can't run concurrent `query()` sessions** —
  parallel calls collide and all-but-one fail instantly. It's also slow (~150s/call). So BCM deep
  dives DEFAULT to a fast parallel **provider** pass (`providerDeepDive`), with the Agent-SDK
  explorer opt-in + serial (`BCM_DEEPDIVE_AGENT=1`). Same lesson applies anywhere you'd fan out
  agents. (The agent-verifier runs one finding at a time, so it's unaffected.)
- **File selection must reserve budget for signal files.** The cluster/keyword selector will
  saturate the file cap and starve the security-critical config/service layer. `robust-scan.js`
  seeds the signal/service-def/config files FIRST, then clusters fill the remainder.
- **Git submodules are not auto-recursed.** For a repo with `.gitmodules`, a plain clone leaves
  submodule dirs empty and their code is silently unscanned. Clone manually with
  `git clone --recurse-submodules` (rewrite `git@`→`https` + token for private submodules) and
  scan via `--repo-dir`. The integrity gate flags this, but does not fix it.
- **Empty default branch (content on a non-default branch).** Some repos keep the real code on a
  non-default branch while `master`/`main` is a bare init (e.g. ServiceNow update-set repos on
  `sn_instances/*`, env branches). A `--single-branch` clone yields an empty tree → the whole scan
  silently sees 0 files → everything rolls up as `not_scanned`. **Fixed:** both `fleet-worker.js` and
  `robust-scan.js` clone, then if the working tree is empty, enumerate branches (cheap `blob:none`
  fetches) and check out the fullest one. Genuinely-empty repos (0 KB, no branches) correctly stay
  empty. This was the root cause of a large batch of falsely "unscanned" repos in an early fleet run.
- **Semgrep rule-pack names are fragile.** An invalid pack (e.g. `p/spring`) makes semgrep exit
  non-zero and emit *nothing* for the whole run. Rule packs live in `LANG_TO_RULESETS` in
  `semgrep-runner.js`; the runner now falls back to `p/security-audit` on error.
- **`dependency_health_score` = 10 can mean "no manifest found", not "clean".** Projects using
  Eclipse `.classpath`, Ant/Ivy, or vendored jars have no manifest the OSV scanner can parse.
  The gate reports this as UNKNOWN; don't read the score as "no vulnerable dependencies".
- **`combined_risk_score` is exploitability-anchored** (`lib/security/finding-scoring.js`,
  `combineRiskFromSignals`, pure + tested). The base comes from the strongest DEMONSTRATED exposure
  (a production critical with a kill chain / external reachability = 8; production criticals with no
  demonstrated path = 6; warnings lower; test/ci/tooling findings don't count as attack surface),
  with small bounded additions for unauthenticated access / internet exposure / PII / vulnerable
  deps, and downweights only for Tolerate/Eliminate/archived. **10 is reserved for the "exposed
  database with no password" archetype.** If you change the scoring, re-score existing rows (loop
  `computeCombinedRisk`+`updateCombinedRisk`) — the score is persisted in `scan_metrics`. A prior
  version summed loosely-scaled gaps × a "Modernize" multiplier and pinned ~half the portfolio at 10
  (a contributor-risk scale bug alone maxed clean apps); avoid reintroducing unbounded additive terms.
- **Verify AI findings before trusting them.** The generator over-classifies; always open the
  actual file/line before affirming a finding is real. The verifier + consensus exist for this.
- **Reports are confidential** (embed real code + any exposed secrets) — `reports/` is gitignored.
- **Adding a reachability framework takes THREE edits, not one.** A per-framework analyzer in
  `lib/reachability/<fw>.js` does nothing unless (1) `framework-detect.js` emits its slug and
  (2) `index.js` maps that slug in `FRAMEWORK_HANDLERS`. The Spring analyzer shipped wired in
  (2) but missing from (1), so Spring apps silently got 0 entry points until detection was added.
  When you add a framework, add a detection test in `tests/reachability-detect.test.js`.
- **Never hand a bare 0–1 fraction to an LLM prompt (or a human-facing string) as a percentage.**
  A `1.0` coverage fraction was once misread by the model as ~1%. Convert with
  `toPercent`/`pctLabel` from `lib/util/format.js` and label the unit (`files_..._percent`,
  `"…%"`). This is covered by a regression test in `tests/integrity-gate.test.js`.
- **Reports must stay self-contained** (no CDN, no external requests — they're confidential). The
  architecture diagram is therefore *pre-rendered* to inline SVG at report time via the Chrome
  binary + the vendored `lib/reporting/vendor/mermaid.min.js` (MIT), not loaded from a CDN. If you
  bump mermaid, keep the global-resolution logic in `mermaid-render.js` in sync (v11 exposes
  `__esbuild_esm_mermaid_nm.mermaid`, not `window.mermaid`).

## Running
```bash
npm run migrate                                   # apply DB migrations (idempotent)
npm test                                          # node:test suite (no ambient ORG_PROFILE)
ORG_PROFILE=alberta npm start                     # dashboard on :3000
npm run robust-scan -- --repo owner/name          # full multi-detector scan of one repo
npm run portfolio-scan -- --concurrency 5         # whole active portfolio (resumable)
npm run report -- --repo owner/name               # standalone HTML + PDF report (per app)
npm run report -- --repo owner/name --chief-architect   # force the rebuild proposal + estimate + business case
npm run portfolio-report -- --repos owner/a,owner/b --title "My System"   # multi-repo ministry meta-report (no rescan)
npm run portfolio-report -- --families families.json --title "My Portfolio"  # group repos into application families (prime + supporting)
```
```bash
npm run bcm-scan -- --repo owner/name         # Business Capability Map: full walk + capability map + deep-dive rebuild specs
npm run bcm-scan -- --repo owner/name --max   # deepest rebuild-grade extraction (raises deep-dive cap to 200 + Agent-SDK explorer)
npm run bcm-register -- --out register.html    # whole-of-gov capability register (cross-repo, consolidation candidates, dup detection)
npm run bcm-register -- --merge "from" "to"    # governance: merge two canonical capabilities
npm run reset-bcm -- --yes                      # rebuild the register from a clean baseline
```
BCM deep dives (`lib/bcm/deep-dive.js`) extract screens/workflows/functions PLUS the rebuild
primitives — `data_entities` (with fields + relationships), `business_rules` (testable
Given/When/Then), `api_endpoints`, `acceptance_criteria` — stored in `file_capability_map.functions`.
From these the report synthesizes (deterministically, no extra LLM) a **data-model ER diagram**
(`lib/reporting/er-diagram.js` → Mermaid `erDiagram` → inline SVG) and an **aggregated API surface**
table (the OpenAPI-first contract surface). `--max` runs the deepest extraction (deep-dive cap 200
+ Agent-SDK explorer). NOTE: a persisted import-graph component diagram remains deferred — it needs
a scan-pipeline change to store the graph; the architecture diagram already gives a component view. The report (`report-html.js`)
opens with a **disposition recommendation** (`lib/reporting/disposition.js` — Remediate/Rebuild/
Retire/Maintain from TIME + security + deps + maintenance + legacy + capability duplication +
modernization cost) and surfaces maintenance/velocity, a tech-modernization profile, the AI-drafted
README, and per-capability security risk. The whole-of-gov register lives in `business_capabilities`
+ `v_bcm_capabilities`; `reset-bcm` clears it, `bcm-register` reports + merges it.

Useful flags: `--repo-dir <path>` (scan a local clone), `--deep` / `--max-fidelity` (see below),
`--skip-semgrep`/`--skip-ai`/`--skip-verifier`,
`--skip-integrity` (disable the gate's LLM pass — deterministic checks still run), `--keep-clone`,
`--dry-run`. Env: `SEMGREP_BIN`, `VERIFIER_MODEL_TIER` (`fast`|`deep`), `INTEGRITY_MODEL_TIER`,
`AGENT_VERIFIER`, `PDF_CHROME_BIN`. See `docs/CONFIGURATION.md` for the full list.

### Deep / max fidelity (one-flag deep scan)

A "deep" scan is the **same pipeline** with the depth caps raised so the AI reads far more of the
tree and cross-checks findings across multiple samples. Rather than the old manual env incantation
(`SCAN_MAX_TOTAL_MB=64 SCAN_MAX_KEY_FILES=5000 SCAN_MAX_FILE_KB=400 CLUSTER_LEFTOVER_MULTIPLIER=40
CONSENSUS_SAMPLES=2 CONSENSUS_MIN=2 VERIFIER_MODEL_TIER=deep`), pass **`--deep`** (or `--max-fidelity`
for the deepest preset), or set `SCAN_FIDELITY=deep|max`. The presets live in `lib/scan/fidelity.js`,
which is **imported first** by `robust-scan.js` so it mutates `process.env` before any module reads
its `SCAN_MAX_*` / `CLUSTER_*` / `CONSENSUS_*` consts. **Explicit env vars always win** over the
preset (it only fills in values you did not set). `portfolio-scan`/`fleet` subprocesses inherit the
env, so `SCAN_FIDELITY=deep npm run portfolio-scan` runs the whole portfolio deep.

## Chief Architect — rebuild proposal (report section "3f")

When a report's disposition is **Rebuild**, `generate-report.js` runs a deep-tier (Opus) pass
(`lib/reporting/chief-architect.js`) that turns the org's **opinionated standards doc** + the app's
already-extracted signals (disposition, architecture, BCM rebuild primitives, security, deps,
maturity) into a concrete rebuild proposal: an executive summary, a **1–2 page migration plan**
(phased strangler-fig, markdown), a **1–2 page future-architecture proposal** (target state mapped
onto the standard stack, markdown), a **target-state Mermaid diagram** (pre-rendered to inline SVG
like the architecture diagram), a **legacy→target component mapping**, a **12-item
rebuild-readiness checklist**, an effort estimate, and key risks. Persisted to
`chief_architect_plans` (migration 27), keyed by `(repository_id, scan_id)`.

- **Standards doc:** an org-authored markdown file loaded by `loadStandardsDoc()` in
  `lib/org-profile.js`, precedence `--standards <path>` > `STANDARDS_DOC` env >
  `profiles/<org>.standards.md` > `profiles/default.standards.md` (generic template;
  `alberta.standards.md` is a worked example). The more prescriptive the doc, the more concrete the
  proposal. Everything is rendered self-contained (markdown via `marked`, diagram via vendored
  mermaid) — no CDN.
- **Report flags:** `--chief-architect` (force even when not Rebuild), `--no-chief-architect`
  (suppress), `--refresh-architect` (recompute a persisted plan), `--standards <path>`.
- **Env:** `CHIEF_ARCHITECT_MODEL_TIER` (default `deep`), `CHIEF_ARCHITECT_OUTPUT_TOKENS`
  (default 12000). Report generation never blocks on it — failures degrade to a skipped section.

## Bottom-up rebuild estimate (report)

The Chief Architect computes a **bottom-up estimate** from the app's ACTUAL extracted unit counts
(services/capabilities, screens, workflows, API integrations, data entities, database) times the
per-unit rates in an org **estimation-heuristics doc** — not a T-shirt guess. Rendered as an
auditable line-item table (`estimateBlock` in `report-html.js`); the one-line `effort_estimate`
headline matches the totals.
- **Doc:** `loadEstimationDoc()` in `lib/org-profile.js`; precedence `--estimation <path>` >
  `ESTIMATION_DOC` > `profiles/<org>.estimation.md` > `profiles/default.estimation.md`. The rates
  already assume an AI-assisted team (a "do not double-discount" rule + a pre-AI adjustment factor
  are baked into the doc).
- **Env:** `CHIEF_ARCHITECT_MODEL_TIER`, `CHIEF_ARCHITECT_OUTPUT_TOKENS`. Persisted to
  `chief_architect_plans.estimate` (migration 28).
- **Database-centric repos (stored-procedure over-count guard).** A PL/SQL / T-SQL / SSIS
  database-scripts repo makes the BCM emit one "capability" (and often a "workflow") per stored-proc
  package. Those are DATA-LAYER logic, not independent application services; pricing 200+ procs as
  200+ full service builds massively over-counts (a 2 MB PL/SQL repo once estimated at $1.4M). When
  `isDatabaseCentric(signals)` is true (`estimate.js`; keyed on the repo's PRIMARY language, so a
  Java/C# app that merely *calls* PL/SQL is NOT tripped), `computeEstimate` prices the service +
  workflow lines as a bounded consolidation/port (`DB_LOGIC_FACTOR`, 0.2) while keeping the
  data-entity + database-migration lines (the real data work) at full weight. Deterministic; tested
  in `tests/estimate.test.js`.

## Business case (report section "Business case", Five Case Model, DM-ready)

A separate deep-tier pass (`lib/reporting/business-case.js`) writes an executive, Deputy-Minister-ready
business case: metadata header, executive "recommendation and ask", the opportunity, **critical
business capabilities** (what each does, who relies on it, consequence if lost), strategic alignment,
a four-option short-list (status quo / remediate-AI-Garage / rebuild-AI-Factory / procure-RFP), **cost
of doing nothing**, benefits + measures, TCO, and a recommendation. It translates technical findings
into service/risk/cost terms (no CVE/framework jargon). The Rebuild option's cost + timeline are
**locked to the Chief Architect estimate** (deterministic override) so the two never disagree. Starts
on its own page and is self-contained/extractable.
- **Doc:** `loadBusinessCaseDoc()`; precedence `--business-case-doc <path>` > `BUSINESS_CASE_DOC` >
  `profiles/<org>.business-case.md` > `profiles/default.business-case.md` (HM Treasury Five Case Model
  + do-nothing baseline + benefits realization).
- **Flags:** `--business-case` (force), `--no-business-case` (suppress); runs alongside the Chief
  Architect otherwise, recomputed with `--refresh-architect`.
- **Env:** `BUSINESS_CASE_MODEL_TIER`, `BUSINESS_CASE_OUTPUT_TOKENS` (default 20000). Persisted to
  `chief_architect_plans.business_case` (migration 29).

## Companion profile docs (org-editable, drive the LLM passes)

All are markdown, resolved by the same precedence (`--flag` > `ENV` > `profiles/<ORG_PROFILE>.<suffix>`
> `profiles/default.<suffix>`). `default.*` is the generic template; `alberta.*` is a worked example.

| Doc | Suffix | Drives | Flag / env |
|---|---|---|---|
| Engineering standards / target stack | `standards.md` | Chief Architect rebuild proposal | `--standards` / `STANDARDS_DOC` |
| Rebuild estimation heuristics | `estimation.md` | Bottom-up estimate | `--estimation` / `ESTIMATION_DOC` |
| Business case template (Five Case Model) | `business-case.md` | Business case | `--business-case-doc` / `BUSINESS_CASE_DOC` |
| Technology vintage table | `tech-vintage.md` | DataPack tech-vintage bands (`lib/reporting/tech-vintage.js`) | `--estimation` n/a; `loadTechVintageDoc` |
| Ministry -> generic portfolio map | `ministries.md` | DataPack ministry alignment (`lib/reporting/ministry-map.js`) | `loadMinistriesDoc` |
| De-identification deny list | `deny-tokens.txt` | Proper nouns/codenames scrubbed from BCM labels + DataPack (`loadDenyTokens`, unioned with `default.deny-tokens.txt`) | `--deny-tokens` > `DENY_TOKENS` |

`deny-tokens.txt` is **plain text** (one token/phrase per line, `#` comments), not markdown, and the
org-specific file is **gitignored** (it names internal systems) - only `default.deny-tokens.txt`
ships. `loadDenyTokens()` always unions the shipped generic base with the org file. Full profile guide
for adopters: [profiles/README.md](profiles/README.md).

Plus one **generated** (not authored) per-org artifact: `profiles/<org>.derived-domains.json` is the
frozen tier-1 domain vocabulary written by `bcm-derive-domains.js` (gitignored; regenerated by the
pipeline, kept for reproducibility). Scan-side exact versions for tier-B vintage are extracted by
`lib/scan/tech-versions.js` into `scan_metrics.tech_versions`.

## `scripts/generate-report.js` — all flags

`--repo owner/name` (or `--last N` / `--all`); `--out <dir>` (default `reports/`); `--no-pdf`;
`--no-evidence` (skip GitHub code fetches); `--concurrency N`. Rebuild-decision artifacts:
`--chief-architect` (force the proposal + estimate + business case even when disposition is not
Rebuild), `--no-chief-architect`, `--no-business-case`, `--refresh-architect` (recompute persisted
plan/business case), `--standards <path>`, `--estimation <path>`, `--business-case-doc <path>`.
The Chief Architect + business case otherwise auto-run only when the disposition is Rebuild.

## Report-time DB tables / migrations added

- 27 `chief_architect_plans` — rebuild proposal (migration plan, future architecture, target
  mermaid, component mapping, readiness checklist, effort, risks). +28 `estimate`/`estimation_source`.
  +29 `business_case`/`business_case_source`.
- 30 `portfolio_analyses` — cached ministry meta-analysis, keyed by `group_key` (repo-set hash).
- 31 `fleet_runs` / `fleet_jobs`; 32 `v_fleet_results` view (whole-org campaign).
- 33 `export_repo_map` (repo → export UUID). 34 `business_capabilities.generic_*` (additive generic
  abstraction). 35 `bcm_taxonomy_baseline` (frozen-taxonomy provenance). 36 `scan_metrics.tech_versions`
  (exact target-framework/versions for tier-B vintage).

## House writing style

`lib/util/style.js`: `STYLE_GUIDE` (appended to LLM prompts) + `enforceStyle()` (deterministic
sanitizer). Reports use **no em/en dashes** (commas/colons; hyphen for numeric ranges);
`finalizeReportStyle()` runs one pass over the whole assembled report HTML, preserving the lone `—`
"no data" placeholder. Markdown is rendered with `marked` server-side (self-contained; no CDN).

---
> Source: [GovAlta/GIT-INSIGHTS-HUB](https://github.com/GovAlta/GIT-INSIGHTS-HUB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->

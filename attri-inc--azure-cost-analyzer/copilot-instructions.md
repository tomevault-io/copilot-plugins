## azure-cost-analyzer

> Analyzes Azure resource group costs, identifies optimization opportunities, and generates professional FinOps reports. This skill should be used when the user wants to analyze Azure resource group spending, find orphaned resources, get cost optimization recommendations, or generate cost analysis reports. Invoked with a resource group name and optional region.


# Azure Resource Group Cost Analyzer

Analyze an Azure resource group to produce a comprehensive cost optimization report following the Microsoft Well-Architected Framework (Cost Optimization pillar) and FinOps Foundation best practices.

This skill is built to be a **single source of truth** for a resource group: it tells the *complete* cost story and is engineered to **never silently skip a resource**. A guaranteed-completeness engine (enumerate → coverage-matrix → reconcile) cross-checks **every ARM-tracked resource type** in the resource group and routes anything unmatched to a generic fallback, so no resource type is silently dropped. (Costs with no RG-scoped ARM object — inter-region/egress bandwidth, subscription-scoped Marketplace SaaS, classic ASM resources — are surfaced separately via Cost Management actuals, not the type reconcile.)

**Arguments:**
- `$0` - Resource Group name (required)
- `$1` - Region for pricing context (optional, defaults to auto-detect from RG)

**Output:** `~/azure-cost-analysis/{rg-name}-cost-analysis.md`

---

## Core Principles (read first)

1. **Guaranteed completeness — no silent drops.** Every distinct ARM type in the RG is either analyzed by a specialized agent (via [`references/resource-coverage-matrix.md`](./references/resource-coverage-matrix.md)) or by the generic fallback. The run is only COMPLETE when it can attest `N of N resource types analyzed, 0 gaps`.
2. **Cost is never null (Cost Resolution Chain).** Every resource MUST end with a cost figure **and a stated basis** — never blank, never an unexplained zero-dollar figure. Resolve in order: **(1) Billed** — Cost Management amortized cost, *only when it returns a real non-zero amount*; **(2) List-price (Retail API)** — when billed is zero/unavailable, compute `usage quantity × current unit rate` from the Azure Retail Prices API (region-correct, priced as of the run date); **(3) List-price (web)** — when the Retail Prices API has no match (Marketplace/third-party/preview/SKU gaps), WebSearch today's price, compute, and cite URL + date; **(4) Manual** — only if no price exists anywhere, flag `price unavailable — manual review`. **Credit/sponsorship rule:** on credit/sponsorship/free-trial/MSDN subscriptions Cost Management commonly reports zero because credits cover spend — a **zero billed amount does NOT mean free**, so the **list-price model becomes the authoritative headline cost** (what the workload would cost at PAYG rates / draws down from credits). [`references/pricing-reference.md`](./references/pricing-reference.md) is an offline fallback to the live Retail Prices API.
3. **Rate optimization is first-class.** Reserved Instances, Savings Plans, Azure Hybrid Benefit, and Spot are often the single largest savings lever — always analyze both "should buy" and "bought-but-wasting" signals (see [`references/commitment-discounts.md`](./references/commitment-discounts.md)).
4. **Safety-first.** Never recommend a destructive change without the protocol in [`references/safety-checklist.md`](./references/safety-checklist.md); distinguish reversible rate changes from destructive deletes.
5. **Proof-gated actions — default to INVESTIGATE.** No destructive action (DELETE / downgrade / redundancy- or retention-reduction) may appear in the report as actionable unless the **verification gate** ([`scripts/verify-findings.sh`](./scripts/verify-findings.sh), Phase 6) has independently re-proven it safe against live Azure. Anything not `VERIFIED_SAFE` is auto-demoted to **INVESTIGATE** with its evidence. The worst case the user can act on is "investigate" — never a wrong delete. (PE/connectivity deletes cap at NEEDS_CONFIRMATION by design — external references can't be proven absent.)
6. **Adaptive & cross-referenced.** Branch on the resource types actually discovered; map dependencies (App Service → Redis, Web App → Storage, PE → Private DNS) before recommending removal.
7. **Actionable.** Every finding ships the exact `az` command to remediate it, with projected savings and a risk level.

---

## Workflow

### Phase 1: Pre-flight Validation

Run the pre-flight check script to validate the environment and capabilities:

```bash
bash scripts/az-preflight-check.sh "$0"
```

The script validates Azure CLI, login, subscription, RG existence, the `resource-graph` extension, and the reachability of Cost Management (ActualCost + AmortizedCost), Azure Retail Prices API, Advisor, and the reservation/savings-plan recommendation APIs. It also emits the resolved region, subscription id, currency/cloud, the **distinct resource-type count**, and writes the authoritative discovered-type set to `~/azure-cost-analysis/$0-discovered-types.txt` and the kind-discriminated set to `~/azure-cost-analysis/$0-discriminated-kinds.txt`. The script always emits one JSON line (an `EXIT` trap guarantees output even on an unexpected abort).

**Required RBAC** (least privilege): `Reader` on the RG (inventory + metrics), `Cost Management Reader` (actual/amortized cost — *not* "billing-reader"), Advisor is covered by Reader, and `Billing Reader` at billing scope for reservation/savings-plan recommendations. Missing optional roles degrade gracefully.

If pre-flight fails on a hard check, report the specific failure and stop. Warn-level checks are **non-fatal**: proceed in degraded "estimate-only" mode and clearly flag which live data was unavailable.

**Agent availability:** this skill ships its analyst subagent at [`.claude/agents/azure-infra-engineer.md`](./.claude/agents/azure-infra-engineer.md). Confirm the `azure-infra-engineer` type resolves before Phase 3; if Task fan-out is unavailable, fall back to running the five lanes sequentially yourself (the degraded path), preserving the same per-lane output files.

### Phase 2: Resource Discovery & Type Enumeration

Establish ground truth of *what exists* before analyzing anything.

```bash
# SUB_ID comes from the preflight JSON (or: SUB_ID=$(az account show --query id -o tsv))
# --subscriptions is REQUIRED: ARG fans out across ALL accessible subscriptions by
# default, so a same-named RG elsewhere would silently pollute the completeness ground truth.
az graph query -q "Resources | where resourceGroup =~ '$0' | summarize Count=count() by type | order by Count desc" \
  --subscriptions "$SUB_ID" --first 1000 --output json

# Kind for kind-discriminated types (microsoft.web/sites = web vs function vs logic app)
az graph query -q "Resources | where resourceGroup =~ '$0' | where type =~ 'microsoft.web/sites' | distinct kind" \
  --subscriptions "$SUB_ID" --output json

# Full inventory (Name, Type, Kind, SKU, Location, Tags). For >1000 resources, page with --skip-token.
az resource list --resource-group "$0" --output json
```

- The preflight already seeds `~/azure-cost-analysis/$0-discovered-types.txt` (lowercased distinct types) and `~/azure-cost-analysis/$0-discriminated-kinds.txt`; confirm/refresh them. These are the **reconciliation ground truth** used in Phase 5.
- Build a resource inventory table: Name, Type, SKU/Tier, Location, Tags.
- Map each discovered type to its owning agent using [`references/resource-coverage-matrix.md`](./references/resource-coverage-matrix.md).

Use queries from [`references/resource-graph-queries.md`](./references/resource-graph-queries.md) (see the *COMPLETENESS SEED* section).

### Phase 3: Parallel Deep Analysis (4 domain agents)

Spawn **4 `azure-infra-engineer` agents in parallel** via the Task tool (the subagent ships at [`.claude/agents/azure-infra-engineer.md`](./.claude/agents/azure-infra-engineer.md)). Each owns a lane defined in the coverage matrix, analyzes **every discovered type in its lane**, prefers live cost data, and emits **one row per resource instance tagged with its lowercased ARM `type`** (so the Phase 5 gate can verify coverage by code) plus a mandatory **Disposition** (`KEEP`/`OPTIMIZE`/`DELETE`/`ARCHIVE`/`MIGRATE`/`CONSOLIDATE`/`INVESTIGATE`), current/projected/savings cost, the exact `az` command, and a risk level. Pass each agent the resource group name, its owned resource list, and the relevant reference files. Agents are **read-only** — they emit remediation commands, never execute them.

**Agent 1 — Compute, Containers & Commitment Discounts**
Compute and container hosting plus the cross-cutting **rate-optimization layer** it owns end-to-end (RI, Savings Plans, Azure Hybrid Benefit, Spot, ephemeral OS disks) and the subscription-scoped reservation/savings-plan recommendation + utilization pulls. Types: VMs, VMSS, disks, snapshots, availability sets, AKS (+ node pools), Container Apps (+ environments), ACI, App Service Plans, Web/Function/Static apps, Batch, Service Fabric, reservations/savings plans, ACR. References: [`commitment-discounts.md`](./references/commitment-discounts.md), [`azure-cli-commands.md`](./references/azure-cli-commands.md), [`resource-graph-queries.md`](./references/resource-graph-queries.md). Save to `~/azure-cost-analysis/$0-compute-analysis.md`.

**Agent 2 — Data, Storage & Backup**
All persistent data, file/disk-storage tiering, caching, and backup/recovery plus reserved-capacity and redundancy levers. Types: Storage Accounts (blob/file services), SQL DB/elastic pools/MI, PostgreSQL/MySQL/MariaDB, Cosmos DB, Redis/Redis Enterprise, Recovery Services & Backup vaults, NetApp, Data Explorer (Kusto), Data Lake. References: [`azure-cli-commands.md`](./references/azure-cli-commands.md), [`resource-graph-queries.md`](./references/resource-graph-queries.md). Save to `~/azure-cost-analysis/$0-data-analysis.md`.

**Agent 3 — Networking & Data Transfer**
Every `Microsoft.Network/*` and `Microsoft.Cdn/*` resource: SKU/tier inventory, idle/over-provisioned premium gateways and firewalls, orphan detection beyond public IPs, the retirement/migration watchlist, and data-transfer/egress cost estimation. References: [`networking-cost-analysis.md`](./references/networking-cost-analysis.md), [`azure-cli-commands.md`](./references/azure-cli-commands.md), [`resource-graph-queries.md`](./references/resource-graph-queries.md). Save to `~/azure-cost-analysis/$0-networking-analysis.md`.

**Agent 4 — Observability, Security, Integration & AI/ML**
Monitoring/logs economics (Log Analytics commitment tiers, table plans, retention, DCR; App Insights sampling; Sentinel surcharge), alert-rule costs, Key Vault/Managed HSM, all integration platforms, all data-engineering platforms, and all AI/ML services (OpenAI PTU, Cognitive commitment tiers, AML idle compute, AI Search units). References: [`ai-ml-integration-cost-analysis.md`](./references/ai-ml-integration-cost-analysis.md), [`azure-cli-commands.md`](./references/azure-cli-commands.md), [`resource-graph-queries.md`](./references/resource-graph-queries.md). Save to `~/azure-cost-analysis/$0-observability-ai-integration-analysis.md`.

### Phase 4: FinOps, Cost Truth & Governance (5th agent, runs AFTER Phase 3)

Spawn a 5th `azure-infra-engineer` agent **after** the four domain agents complete (it consumes their outputs). It owns:

1. **Ground-truth cost** from the Cost Management Query API (ActualCost + AmortizedCost) per `ResourceId` / `ServiceName` / tag. **Detect credit/sponsorship mode:** if the preflight flagged a credit subscription (quotaId) OR the query returns zero/empty while resources clearly consume, mark billed cost as *credit-covered* and make the **list-price model authoritative** for every resource (apply the Cost Resolution Chain in Core Principle #2 — Retail Prices API, then web search). Every resource ends with a cost + basis; never report a silent zero (a zero-dollar figure with no stated basis).
2. **Azure Advisor** full cost-recommendation catalog + the Advisor Score (Cost) KPI.
3. **Commitment-discount consolidation**: merge the Compute agent's RI/Savings-Plan/AHB findings, flag reservations <90% utilized, and surface buy recommendations.
4. **Governance**: tag audit (untagged/required-tag/inheritance), resource locks, and `resourcechanges` (last 14 days) for cost-spike root cause.
5. **Operate-phase guardrails**: recommend (or create, if asked) budgets, ML anomaly alerts (`scheduledActions` InsightAlert), and a scheduled FOCUS export.
6. **Retirement/deprecation watch** (consolidated, cross-domain).

References: [`finops-framework-and-tooling.md`](./references/finops-framework-and-tooling.md), [`commitment-discounts.md`](./references/commitment-discounts.md). Save to `~/azure-cost-analysis/$0-finops-completeness-analysis.md`.

### Phase 5: Completeness Reconciliation (MANDATORY GATE)

The completeness engine guarantees nothing is missed. **Run the code-enforced gate** — do not eyeball it:

```bash
# Path-independent (resolves matrix-keys.txt relative to the script, not the CWD).
# Exit 0 = "N of N types analyzed, 0 gaps"; exit 1 = unexplained types remain; exit 2/3 = inputs missing.
bash scripts/reconcile.sh "$0"
```

`reconcile.sh` reads `$0-discovered-types.txt`, every `$0-*-analysis.md`, `references/matrix-keys.txt`, and `references/matrix-kind-keys.txt`, and **fails the run** unless every discovered type appears in the analysis output. It also emits kind-routing warnings for `microsoft.web/sites` (web vs function vs Logic App) using `$0-discriminated-kinds.txt`.

- For **every** type the gate prints as a GAP (no specialized rule), run the **generic fallback** (see [`references/resource-coverage-matrix.md`](./references/resource-coverage-matrix.md) → *Generic fallback rule*): pull `EffectiveCost`/`AmortizedCost` by `ResourceId`, resolve a live unit price by `ServiceName` + region, attempt a generic idle heuristic via `az monitor metrics list-definitions`, read tags + locks, and emit a row marked `Analyzed (generic fallback)`. If a type cannot be priced, list it as `Analyzed (generic) — cost unavailable, manual review needed` — **never drop it**, then re-run `reconcile.sh` until it exits 0.
- **Scope caveat:** the gate covers ARM-tracked resources in the RG. Costs with no RG-scoped ARM object (inter-region/egress bandwidth, subscription-scoped Marketplace SaaS, classic ASM resources) are surfaced via the Cost Management actuals in Phase 4, not the type reconcile — call these out explicitly in the report.
- **Invariant:** the count of report rows by type must equal the discovered-type count.
- The run is declared **COMPLETE** only when the residual GAP set is empty, printing the attestation: `N of N resource types analyzed, 0 gaps`. If it is not empty, the report is marked INCOMPLETE and the unmatched types are listed explicitly.

### Phase 6: Evidence-Based Verification Gate (MANDATORY — before any report)

**No destructive recommendation reaches the report unproven.** Each analysis agent emits its proposed actions as JSON lines to `~/azure-cost-analysis/$0-findings.jsonl` (`{"id","action","type","name","resourceId","savings","claim"}`). Then run the gate:

```bash
# Independently re-proves every DELETE/downgrade against live Azure (read-only).
# Exit 0 = all destructive findings VERIFIED_SAFE; exit 1 = some not proven (must demote).
bash scripts/verify-findings.sh "$0"
```

For each destructive finding the gate returns a verdict + evidence (see [`references/verification-checks.md`](./references/verification-checks.md)):
- **`VERIFIED_SAFE`** — all automated dependency proofs passed (e.g. unattached/0-record/no-DNS-ref/reversible). Only these may appear in the report as an actionable DELETE/OPTIMIZE.
- **`NEEDS_CONFIRMATION`** — looks safe but a residual risk can't be auto-proven (e.g. PE deletes — external hardcoded refs; redundancy/retention downgrades — RPO/DR). Strong evidence is attached.
- **`REJECTED`** — a dependency proves it is in use (e.g. public IP attached to a NAT gateway, DNS record resolves to a PE). **Must not be recommended.**

**Auto-demotion rule:** any destructive finding that is not `VERIFIED_SAFE` is **demoted to INVESTIGATE** in the report (with its evidence), never presented as "do this." The report's *DELETE action view* contains **only** `VERIFIED_SAFE` items. Re-run the gate after any demotion; proceed to the report only when it reflects reality.

### Phase 7: Report Compilation

Merge all agent findings into a single professional report using the structure in [`references/report-template.md`](./references/report-template.md):

1. Read all agent outputs from `~/azure-cost-analysis/$0-*-analysis.md` and the verdicts from `~/azure-cost-analysis/$0-verification.jsonl`.
2. Lead with the **Resource Coverage Reconciliation** table (completeness gate) and the **Actual vs Estimated Cost** reconciliation.
3. Apply the **verification verdicts**: only `VERIFIED_SAFE` actions go in the DELETE/OPTIMIZE action views; everything else is INVESTIGATE with its evidence. Tag every action with its verdict + the proof that was run.
4. Cross-reference findings between agents (e.g. App Service → Redis, Web App → Storage, PE → Private DNS).
5. Compute costs from **live** data first; fall back to [`references/pricing-reference.md`](./references/pricing-reference.md) only when live data is unavailable, labeling such numbers as estimates.
6. Generate 3 optimization scenarios (Conservative, Moderate, Aggressive).
7. The report must be a complete **Single Source of Truth** — it MUST include, populated with live data: **every resource itemized individually** (instance-level, not ranges); a **Cost Breakdown (fixed + variable)** where **variable/usage cost is measured** from Azure Monitor metrics (egress, requests, data processed, ingestion) × live rates — not omitted; **Governance** (tag compliance %, untagged list, resource locks, owners); **Azure Advisor** recommendations; the verification verdicts; and the completeness/disposition attestations.
8. Produce the final report at `~/azure-cost-analysis/$0-cost-analysis.md`, then run the **SSOT gate**:

```bash
# Fails unless EVERY resource is itemized and all mandatory SSOT sections are present + populated.
bash scripts/ssot-gate.sh "$0"
```

If the gate exits non-zero, add the missing sections/resources and re-run until it passes. Only a passing SSOT gate means the report is complete.

### Phase 8: Summary & Recommendations

Present to the user:
1. **Completeness attestation** (`N of N resource types analyzed, 0 gaps`).
2. Executive summary: total estimated/actual monthly cost and top 3 savings opportunities (with $ impact).
3. **Rate optimization**: RI/Savings-Plan/AHB opportunities and any wasted (under-utilized) reservations.
4. Critical issues requiring immediate attention (security, expiring certs, governance gaps).
5. **Retirements & migration watchlist** with deadlines.
6. **Operate-phase guardrails** to make savings stick (budgets, anomaly alerts, exports).
7. Recommended next steps gated by [`references/safety-checklist.md`](./references/safety-checklist.md), each with its `az` command and risk level.
8. Link to the full report file.

---

## Reference Files

Load these as needed during analysis:

- **Resource Coverage Matrix**: [references/resource-coverage-matrix.md](./references/resource-coverage-matrix.md) — the never-miss-anything backbone mapping every ARM type to its owning agent, analysis, and levers (+ [matrix-keys.txt](./references/matrix-keys.txt)).
- **Commitment Discounts**: [references/commitment-discounts.md](./references/commitment-discounts.md) — Reserved Instances, Savings Plans, Azure Hybrid Benefit, Spot, and reservation-waste remediation with the recommendation/utilization APIs.
- **Networking Cost Analysis**: [references/networking-cost-analysis.md](./references/networking-cost-analysis.md) — premium-service utilization thresholds, full orphan detection, retirement watchlist, and egress/data-transfer estimation.
- **AI/ML & Integration Cost Analysis**: [references/ai-ml-integration-cost-analysis.md](./references/ai-ml-integration-cost-analysis.md) — OpenAI PTU, Sentinel, Log Analytics table plans, ACR, integration platforms, data-engineering, AML, AI Search.
- **FinOps Framework & Tooling**: [references/finops-framework-and-tooling.md](./references/finops-framework-and-tooling.md) — live Retail Prices API, Cost Management Query/Cost Details/FOCUS, Advisor + score, budgets/anomaly/exports, carbon, FinOps + WAF mapping, retirement register.
- **Azure CLI Commands**: [references/azure-cli-commands.md](./references/azure-cli-commands.md) — all Azure CLI commands organized by resource type.
- **Resource Graph Queries**: [references/resource-graph-queries.md](./references/resource-graph-queries.md) — KQL for completeness seeding and orphaned/zombie/underutilized resource detection.
- **Pricing Reference**: [references/pricing-reference.md](./references/pricing-reference.md) — *offline fallback* Azure pricing estimates by SKU.
- **Report Template**: [references/report-template.md](./references/report-template.md) — professional report structure following FinOps standards.
- **Safety Checklist**: [references/safety-checklist.md](./references/safety-checklist.md) — pre-deletion safety protocol and reversible-vs-destructive guidance.
- **Verification Checks**: [references/verification-checks.md](./references/verification-checks.md) — the per-action safety proofs the Phase 6 gate runs, and how verdicts map to dispositions.

## Scripts

- **Pre-flight Check**: [scripts/az-preflight-check.sh](./scripts/az-preflight-check.sh) — validates Azure CLI, login, subscription, resource group, capability/API reachability; seeds the discovered-type + discriminated-kind sets and `cost_visibility`.
- **Completeness Gate**: [scripts/reconcile.sh](./scripts/reconcile.sh) — Phase 5; proves every discovered type was analyzed (`N of N, 0 gaps`), kind-aware.
- **Verification Gate**: [scripts/verify-findings.sh](./scripts/verify-findings.sh) — Phase 6; independently re-proves every destructive finding against live Azure and auto-demotes anything not `VERIFIED_SAFE`.
- **SSOT Gate**: [scripts/ssot-gate.sh](./scripts/ssot-gate.sh) — Phase 7; fails unless the final report itemizes **every** resource and contains all mandatory SSOT sections (governance/tags, locks, Advisor, fixed+variable cost, verification, attestations).

---

## Key Design Principles

1. **Guaranteed Completeness**: enumerate-then-reconcile with a generic fallback; no resource type is ever silently skipped (attest `N of N, 0 gaps`).
2. **Live Cost Truth over static estimates**: actual billed cost (Cost Management) and live unit rates (Retail Prices API) take precedence; the static pricing table is a fallback only.
3. **Adaptive Analysis**: dynamically adjust analysis based on discovered resource types — never assume a fixed resource list.
4. **FinOps-Aligned**: follow the Microsoft Well-Architected Framework Cost Optimization pillar and FinOps Foundation phases (Inform/Optimize/Operate).
5. **Rate Optimization First-Class**: surface both buy (RI/SP/AHB/Spot) and waste (under-utilized reservations) signals.
6. **Safety-First**: never recommend hard deletes without the safety checklist; distinguish reversible rate changes from destructive deletes; recommend resource locks before deletion.
7. **Parallel Execution**: 4 domain agents run in parallel; the FinOps/completeness agent runs last to consolidate.
8. **Cross-Reference Dependencies**: map connections between resources before recommending deletions.
9. **Actionable Output**: every recommendation includes an `az` CLI command, projected savings, and a risk level.

---
> Source: [Attri-Inc/azure-cost-analyzer](https://github.com/Attri-Inc/azure-cost-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

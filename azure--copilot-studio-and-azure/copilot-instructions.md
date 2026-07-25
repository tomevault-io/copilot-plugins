## copilot-studio-and-azure

> > Machine-readable guide for AI coding agents (GitHub Copilot, Agent Forge / FAF) working with this accelerator.

# AGENTS.md — Azure & Copilot Studio Pricing Skill

> Machine-readable guide for AI coding agents (GitHub Copilot, Agent Forge / FAF) working with this accelerator.
> The **FAF Evaluation Metadata** section maps this accelerator to the Implementation Recommender Agent's 8 scoring areas so it can be scored automatically as an implementation candidate.

## What this accelerator is

A **GitHub Copilot skill** (`SKILL.md` + reference files) that lets Copilot Chat fetch **real-time Azure retail pricing** from the public [Azure Retail Prices API](https://prices.azure.com) and **estimate Copilot Studio agent credit consumption** — directly inside VS Code, with no Azure subscription or authentication required.

> The pricing output is an **indicative estimate only** — it does not represent actual consumption.

Capabilities:
- Live Azure pricing lookups by service, SKU, and region.
- Monthly/annual cost estimates for common workloads (VMs, Functions, Storage, Cosmos DB, AKS, Azure OpenAI, and more).
- Copilot Studio credit/USD consumption estimation from usage patterns.

## Project structure

| Path | Purpose |
|---|---|
| `README.md` | Setup (repo-level, user-level, or install-from-GitHub) + example prompts |
| `SKILL.md` | Core skill definition — triggers, API usage, instructions *(installed under `.github/skills/azure-pricing/`)* |
| `references/COPILOT-STUDIO-RATES.md` | Copilot Studio billing rates + estimation formulas (fallback) |
| `references/COST-ESTIMATOR.md` | Azure cost formulas by service type |
| `references/REGIONS.md` | Region display name → `armRegionName` mapping |
| `references/SERVICE-NAMES.md` | Case-sensitive `serviceName` values for API filters |

> Note: the skill files live in the repo's `.github/skills/azure-pricing/` (and `skills/azure-pricing/`) folders. This accelerator folder documents and packages the skill.

## How an agent should work with it

- This is a **prompt/skill asset, not deployable code** — no build, no infra.
- To activate: copy `azure-pricing/` into `.github/skills/` (repo-level) **or** the VS Code user `skills/` folder, then invoke via natural-language pricing questions in Copilot Chat.
- Requires VS Code 1.99+ with GitHub Copilot Chat and internet access to `prices.azure.com` and `learn.microsoft.com`.
- The Azure Retail Prices API is **public and unauthenticated** — do not add credentials.
- When editing, keep `serviceName` values case-sensitive per `references/SERVICE-NAMES.md` and region values aligned to `references/REGIONS.md`.

## Setup / deploy commands

No CLI deploy. Copy the skill folder into `.github/skills/azure-pricing/` (or the user-level skills directory) and restart/refresh Copilot Chat.

---

## FAF Evaluation Metadata

Consumed by the FAF **Implementation Recommender Agent** (`accelerators-scoring-card.md`). Values describe what this accelerator *already provides*.

```yaml
accelerator:
  name: "Azure Copilot Pricing Skill"
  slug: "azure-copilot-pricing"
  repo_path: "accelerators/azure-copilot-pricing"
  maturity: "sample"
  primary_platform: "GitHub Copilot (VS Code skill)"

use_case:
  primary_scenario: "In-editor Azure retail pricing lookups + Copilot Studio credit estimation"
  features:
    - "Live Azure Retail Prices API queries"
    - "Monthly/annual cost estimation formulas per service"
    - "Copilot Studio credit + USD consumption estimation"
  domain: "developer tooling / cost estimation (FinOps design-time)"
  extensibility: "Add reference tables / service formulas in references/"

agentic_patterns:
  orchestration: "single skill invoked by GitHub Copilot Chat"
  tool_function_calling: "REST call to Azure Retail Prices API via skill instructions"
  memory_state: "none"
  human_in_the_loop: "conversational (developer in the loop)"
  guardrails: "explicit 'indicative estimate only' disclaimer"

data_scenario:
  data_source_types: ["public REST pricing API"]
  connectors: ["Azure Retail Prices API (prices.azure.com)", "learn.microsoft.com"]
  retrieval_patterns: ["live API fetch", "cached reference tables fallback"]
  transformation_pipelines: "cost estimation formulas"

ai_stack:
  llm_provider: "GitHub Copilot (host model)"
  search_integration: "none"
  agent_framework: "GitHub Copilot skills"
  observability: "n/a"
  languages: ["Markdown skill definition", "REST"]

hosting:
  compute_platform: "local VS Code (no cloud hosting)"
  scaling_model: "n/a"
  regional_availability: "n/a (client-side skill)"

api_integration:
  internal_apis: "none"
  external_saas_apis: ["Azure Retail Prices API (public, unauthenticated)"]
  mcp_support: false
  auth: ["none (public API)"]

ui_scenario:
  channels: ["VS Code Copilot Chat"]
  ui_framework: "n/a"
  ux_patterns: ["conversational prompts"]

infrastructure:
  iac: "none"
  cicd: "none (copy skill folder)"
  monitoring: "none"

stack_summary: ["GitHub Copilot Chat", "REST API", "Markdown skill"]
```

---
> Source: [Azure/Copilot-Studio-and-Azure](https://github.com/Azure/Copilot-Studio-and-Azure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->

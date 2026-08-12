## pix-ai-coding-assistant

> This plugin brings vulnerability intelligence into the AI coding agent development loop. Developers get automatic security scanning on commit/install, on-demand vulnerability analysis, exploit intelligence, and AI-driven remediation — all without leaving their editor.

# Vulnetix AI Coding Agent Plugin — Product Requirements

## Purpose

This plugin brings vulnerability intelligence into the AI coding agent development loop. Developers get automatic security scanning on commit/install, on-demand vulnerability analysis, exploit intelligence, and AI-driven remediation — all without leaving their editor.

## System Context

```mermaid
graph LR
    subgraph "Developer Machine"
        CC["Claude Code"]
        PLG["pix-ai-coding-assistant<br/>(this repo)"]
        CLI["Vulnetix CLI<br/>(../cli)"]
        CC --> PLG
        PLG --> CLI
    end

    subgraph "Vulnetix Cloud (AWS)"
        API["VDB API<br/>(../vdb-api)<br/>Vulnerability data"]
        SITE["VDB Site API<br/>(../vdb-site)<br/>Auth & accounts"]
        DB[(PostgreSQL 18<br/>VDB + Accounts)]
        API --> DB
        SITE --> DB
    end

    subgraph "Infrastructure"
        MGR["VDB Manager<br/>(../vdb-manager)<br/>Terraform IaC"]
        MGR -.->|deploys| API
        MGR -.->|deploys| SITE
        MGR -.->|provisions| DB
    end

    CLI -->|"GET /v1/vuln/*<br/>GET /v2/vuln/*"| API
    CLI -->|"POST /v1/cli/device/*<br/>GET /v1/account"| SITE

    subgraph "Third Party"
        GH["GitHub API<br/>Dependabot · CodeQL"]
        STRIPE["Stripe<br/>Subscriptions"]
    end

    PLG --> GH
    SITE --> STRIPE
```

## Repository Dependencies

### Vulnetix CLI (`../cli`)

The plugin's runtime dependency. Every skill and command shells out to CLI subcommands.

| CLI Command | Used By | VDB API Endpoint |
|-------------|---------|------------------|
| `vulnetix vdb vuln <id>` | vuln, exploits, fix, remediation skills | `GET /v1/vuln/{id}` |
| `vulnetix vdb vulns <pkg>` | vuln skill (package mode) | `GET /v1/{package}/vulns` |
| `vulnetix vdb metrics <id>` | vuln, exploits skills | `GET /v1/vuln/{id}` (metrics) |
| `vulnetix vdb exploits <id>` | exploits skill | `GET /v1/exploits/{id}` |
| `vulnetix vdb fixes <id>` | fix skill | `GET /v1/vuln/{id}/fixes` |
| `vulnetix vdb exploits-search` | exploits-search skill, vdb-exploits-search cmd | `GET /v1/exploits` |
| `vulnetix vdb remediation <id>` | remediation skill, vdb-remediation cmd | `GET /v2/vuln/{id}/remediation-plan` |
| `vulnetix auth status` | hooks (auth check) | — |
| `vulnetix env` | context enrichment | — |

The CLI handles authentication (API key via `vulnetix auth login`), request signing, and JSON response parsing.

### VDB API (`../vdb-api`)

The vulnerability data backend. Aggregates 12+ sources (CVE.org, NVD, VulnCheck, CISA KEV, GitHub/OSV, EUVD) into enriched CVE 5.1 responses.

**Key capabilities the plugin relies on:**
- Vulnerability lookup with CVSS v2/v3/v4, EPSS, KEV enrichment
- Exploit records with source attribution (ExploitDB, Metasploit, Nuclei, etc.)
- Fix intelligence with registry versions, distribution patches, source fixes
- V2 remediation plans with workarounds, advisories, CWE guidance, verification steps
- Package vulnerability listings with affected version ranges
- SBOM/manifest scanning endpoints

**API versions:** V1 (original endpoints), V2 (discrete remediation, scanning, cloud locators, malware detection)

**Authentication:** CLI exchanges org credentials at VDB Site API for JWT, then uses API key (`ApiKey {orgId}:{hexDigest}`) for VDB API requests.

### VDB Site (`../vdb-site`)

Account management backend. The plugin depends on it for:

- **Authentication:** `POST /v1/cli/device/authorize` + `/v1/cli/device/token` — RFC 8628 device grant; the user approves in a browser signed in to the Vulnetix identity provider, and the CLI receives an API key
- **Account info:** `GET /v1/account` — subscription tier, rate limits
- **Usage tracking:** `GET /v1/usage` — request counts, audit log
- **Sign-up:** browser only, at https://www.vulnetix.com/vdb-register (Community tier, free). There is no unauthenticated endpoint that mints credentials from an email.

**Subscription tiers affecting plugin behavior:**
| Tier | Requests/Day | Cost |
|------|-------------|------|
| Community | 50 | Free |
| Pro | 2,000 | $20/mo |
| Teams | 100,000 | $250/mo |
| Enterprise | Custom | Custom |

Rate limit exhaustion causes CLI commands to fail with 429 errors. The session-summary hook could surface this.

### VDB Manager (`../vdb-manager`)

Terraform IaC that provisions and deploys everything the plugin talks to:

- **ECS Fargate** — runs VDB API and Site API containers
- **PostgreSQL 18 (RDS)** — stores vulnerability data and accounts with read replicas
- **Secrets Manager** — JWT secrets, API keys, Stripe keys shared across services
- **Stripe products** — subscription tier definitions (prices, limits)
- **Lambda functions** — scheduled data processing (source ingestion, EPSS updates)
- **S3** — artifact storage for SBOMs and scan results
- **CI/CD** — GitHub Actions with AWS OIDC for deployment

Changes to rate limits, tiers, or API infrastructure are made here, not in the API repos directly.

## Plugin Feature Matrix

### Skills — LLM-Guided Workflows

| Skill | Reads Memory | Writes Memory | Uses CLI | Uses GitHub | Modifies Code |
|-------|-------------|---------------|----------|-------------|---------------|
| dashboard | Yes | No | No | No | No |
| vuln | Yes | Yes | `vdb vuln`, `vdb vulns`, `vdb metrics` | Dependabot alerts | No |
| exploits | Yes | Yes | `vdb exploits`, `vdb metrics` | — | No |
| exploits-search | Yes | Yes | `vdb exploits-search` | — | No |
| fix | Yes | Yes | `vdb fixes`, `vdb vuln` | Dependabot alerts/PRs | Yes (manifests) |
| remediation | Yes | Yes | `vdb remediation` | Dependabot, CodeQL | Yes (manifests + code) |
| package-search | Yes | Yes | `vdb vulns` | — | No |

### Hooks — Event-Driven Automation

```mermaid
graph LR
    subgraph "Session Lifecycle"
        SS["SessionStart"] -->|session-summary.sh| D1["Display vuln status"]
        UP["UserPromptSubmit"] -->|vuln-context-inject.sh| D2["Inject vuln context"]
        ST["Stop"] -->|stop-reminder.sh| D3["Remind open vulns"]
    end

    subgraph "Tool Lifecycle"
        PRE1["PreToolUse:Bash"] -->|pre-commit-scan.sh| D4["Scan packages<br/>before commit"]
        PRE2["PreToolUse:Edit|Write"] -->|manifest-edit-scan.sh| D5["Gate manifest<br/>edits with scan"]
        POST["PostToolUse:Bash"] -->|post-install-scan.sh| D6["SBOM after<br/>npm/pip/go install"]
    end
```

### Commands — Deterministic CLI Wrappers

Commands bypass the LLM (`disable-model-invocation: true`). They run a CLI command, parse JSON, and display structured output. Use these for scripting, quick lookups, or when you want raw data without AI interpretation.

### Agents — Autonomous Multi-Turn

The `bulk-triage` agent runs `/vulnetix:exploits` in parallel across multiple vulnerabilities, then produces a consolidated triage report sorted by CWSS priority. It coordinates memory writes to avoid race conditions.

## Data Architecture

```mermaid
graph TD
    subgraph "Repo Root"
        VD[".vulnetix/"]
        VD --> MY["memory.yaml<br/>Vulnerability state"]
        VD --> SC["scans/<br/>CycloneDX SBOMs"]
        VD --> PC["pocs/<br/>Cached exploit source"]
    end

    subgraph "memory.yaml Sections"
        MY --> MAN["manifests:<br/>Tracked files, scan times"]
        MY --> VUL["vulnerabilities:<br/>Status, decisions, CWSS,<br/>threat models, PoCs"]
    end

    SK["Skills"] -->|read/write| MY
    HK["Hooks"] -->|read/write| MY
    HK -->|generate| SC
    SK -->|cache| PC
```

## Development Guidelines

- **Skill implementations** are `SKILL.md` files with YAML frontmatter (`name`, `description`, `user-invocable`, `allowed-tools`, `model`)
- **Command implementations** are markdown files in `commands/` with `disable-model-invocation: true`
- **Hook scripts** are bash, registered in `hooks/hooks.json`, must exit 0 for informational output or non-zero to block
- **Agent definitions** are markdown files in `agents/` with frontmatter (`name`, `description`, `effort`, `maxTurns`, `allowed-tools`, `model`)
- **Website** is Hugo (Go templates) at `website/` — every registered feature must have a corresponding doc page under `website/content/docs/`
- The canonical schema for `.vulnetix/memory.yaml` is defined in `skills/fix/references/memory-yaml-schema.md`
- All skills share memory — coordinate writes carefully, especially in the bulk-triage agent

### Skill quality gate

Every skill under `vulnetix/skills/` must have:

1. A `## Use when` body section in `SKILL.md`.
2. The phrase "Use when" in its frontmatter `description`.
3. `evals/evals.json` — schema per Anthropic [`schemas.md`](https://github.com/anthropics/skills/blob/main/skills/skill-creator/references/schemas.md); ≥3 evals; each with ≥5 verifiable `expectations` (named CLI calls, output fields, or side-effects — never vague).
4. `evals/trigger-eval.json` — 10 `should_trigger` + 10 `should_not_trigger` queries, per [Sogl, "Skills Without Evals Are Just Markdown and Hope"](https://dev.to/danielsogl/skills-without-evals-are-just-markdown-and-hope-3a71).

SKILL.md structure is enforced locally by `pre-commit` (see `.pre-commit-config.yaml`) and on PRs by `.github/workflows/skill-validator.yml` — both run [`agent-ecosystem/skill-validator`](https://github.com/agent-ecosystem/skill-validator) with `--strict --allow-dirs=evals`.

---
> Source: [Vulnetix/pix-ai-coding-assistant](https://github.com/Vulnetix/pix-ai-coding-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

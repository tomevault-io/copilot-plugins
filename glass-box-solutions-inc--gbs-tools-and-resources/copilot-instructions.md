## gbs-tools-and-resources

> **Unified tools, agents, and resources monorepo — agent services, MCP servers, utilities, reference libraries, and security wordlists.**

# gbs-tools-and-resources

**Unified tools, agents, and resources monorepo — agent services, MCP servers, utilities, reference libraries, and security wordlists.**

---

## ⚠️ CRITICAL GUARDRAILS (READ FIRST)

1. **NEVER push without permission** — Even small fixes require express user permission. No exceptions.
2. **NEVER expose secrets** — No API keys, tokens, credentials in git, logs, or conversation.
3. **NEVER force push or skip tests** — 100% passing tests required.
4. **ALWAYS read parent CLAUDE.md** — `~/CLAUDE.md` for org-wide standards.
5. **ALWAYS use Definition of Ready** — 100% clear requirements before implementation.

---

## Project Overview

This monorepo consolidates 17 Glass Box packages into a single discoverable location. Each package retains its own structure, language, and build tooling — there is no unified build system at the monorepo root.

**Initial consolidation:** 2026-03-01 (7 utility/research repos)
**Agent consolidation:** 2026-03-08 (4 agent packages added)
**Pattern follows:** `internal-tools` (absorbed command-center, glass-box-board, glass-box-hub, Squeegee on 2026-03-01)

---

## Packages

### Agent Services

| Package | Stack | Purpose | Deploy |
|---------|-------|---------|--------|
| [`packages/spectacles/`](packages/spectacles/) | Python 3.12, FastAPI, Playwright, Gemini | Browser automation + documentation intelligence curator | Cloud Run: `glassbox-spectacles` |
| [`packages/merus-expert/`](packages/merus-expert/) | Python 3.12, FastAPI, Claude, Gemini | MerusCase domain agent — 13 tools, SSE streaming | Cloud Run: `merus-expert` |
| [`packages/agent-swarm/`](packages/agent-swarm/) | NestJS 11, TypeScript, Socket.io | DAG-based multi-agent task orchestration | Standalone NestJS library (canonical) |
| [`packages/agentic-debugger/`](packages/agentic-debugger/) | Node.js, Claude Code, GitHub Actions | Automated CI test failure debugging agent | Standalone CI debugging agent (canonical) |

### Operations & Audit

| Package | Stack | Purpose | Deploy |
|---------|-------|---------|--------|
| [`packages/compliance-auditor/`](packages/compliance-auditor/) | Node.js 20, Fastify 5, Prisma, Zod | SOC2 + HIPAA compliance code scanning | Cloud Run |
| [`packages/gbs-integration-validator/`](packages/gbs-integration-validator/) | Node.js 20, Fastify 5, Zod | API integration validation (7 platforms) | Cloud Run |
| [`packages/invoice-reconciliation-tester/`](packages/invoice-reconciliation-tester/) | Node.js 20, Fastify 5, Prisma, Faker | Invoice reconciliation algorithm testing | Cloud Run |

### Utilities & Libraries

| Package | Stack | Purpose |
|---------|-------|---------|
| [`packages/hindsight/`](packages/hindsight/) | Python/FastAPI, Next.js, pgvector, Rust | Agent memory system (retain/recall/reflect) |
| [`packages/mcp-servers/`](packages/mcp-servers/) | Python, Node.js, MCP SDK | MCP server collection (kb-api, kb-db, wc-paralegal, social-media) |
| [`packages/phileas/`](packages/phileas/) | Java 11+, Maven | PII/PHI redaction library (30+ entity types) |
| [`packages/yevrah_terminal/`](packages/yevrah_terminal/) | Python 3.x, Groq, Cohere | Terminal legal research (keyword + semantic search) |
| [`packages/merus-test-data-generator/`](packages/merus-test-data-generator/) | Python 3.12, reportlab, Faker | WC test case generator (10,000+ templated PDFs across 97+ subtypes; AMA Guides 5th Ed. impairment content, 30 edge case scenarios, Browserless MerusCase batch integration) |

### Reference

| Package | Purpose |
|---------|---------|
| [`packages/awesome-agent-skills/`](packages/awesome-agent-skills/) | Curated catalog of 180+ AI agent skills |
| [`packages/awesome-claude-code/`](packages/awesome-claude-code/) | Curated list of skills, hooks, slash-commands, agents, and plugins for Claude Code |
| [`packages/cli-anything/`](packages/cli-anything/) | Claude Code plugin — transform GUI apps into agent-native CLIs (GIMP, Blender, etc.) |
| [`packages/ui-ux-pro-max-skill/`](packages/ui-ux-pro-max-skill/) | AI design intelligence skill for professional UI/UX across platforms |
| [`packages/SecLists-GBS-Branch/`](packages/SecLists-GBS-Branch/) | Security testing wordlists (placeholder) |

---

## Commands

Each package builds independently. See each package's own README.md or CLAUDE.md for build/run/test commands.

### Quick navigation

```bash
# Agent services
cd packages/spectacles/
cd packages/merus-expert/
cd packages/agent-swarm/
cd packages/agentic-debugger/

# Utilities
cd packages/hindsight/
cd packages/mcp-servers/
cd packages/phileas/
cd packages/yevrah_terminal/
cd packages/merus-test-data-generator/
```

### Package-specific tests

```bash
# spectacles (Python)
cd packages/spectacles && pytest

# merus-expert (Python)
cd packages/merus-expert && pytest

# hindsight (Python + Next.js)
cd packages/hindsight && pytest

# yevrah_terminal (Python)
cd packages/yevrah_terminal && pytest

# phileas (Java/Maven)
cd packages/phileas && mvn test

# mcp-servers (varies by server)
cd packages/mcp-servers/servers/[server-name] && npm test
```

---

## Architecture

```
gbs-tools-and-resources/
├── CLAUDE.md                         # This file
├── README.md                         # Human-facing overview
├── .gitignore
└── packages/
    ├── spectacles/                   # Browser automation + curator (Python/FastAPI)
    ├── merus-expert/                 # MerusCase agent (Python/FastAPI)
    ├── agent-swarm/                  # Multi-agent orchestration (NestJS) — standalone library
    ├── agentic-debugger/             # CI debugging agent (Claude Code) — standalone
    ├── compliance-auditor/           # SOC2/HIPAA compliance scanner (Fastify)
    ├── gbs-integration-validator/    # API integration validator (Fastify)
    ├── invoice-reconciliation-tester/ # Invoice reconciliation tester (Fastify)
    ├── hindsight/                    # Agent memory system (Python/FastAPI + Next.js + Rust)
    ├── mcp-servers/                  # MCP server collection (Python + Node.js)
    │   └── servers/
    │       ├── kb-api-mcp/
    │       ├── kb-db-mcp/
    │       ├── social-media-mcp/
    │       └── wc-paralegal-mcp/
    ├── phileas/                      # PII/PHI redaction library (Java/Maven)
    ├── yevrah_terminal/              # Terminal legal research (Python)
    ├── merus-test-data-generator/    # WC test case PDF generator (Python)
    ├── awesome-agent-skills/         # Agent skills catalog (Markdown)
    ├── awesome-claude-code/          # Claude Code ecosystem resource list
    ├── cli-anything/                 # Claude Code plugin — GUI-to-CLI harness (Python/Click)
    ├── ui-ux-pro-max-skill/          # AI design intelligence skill
    └── SecLists-GBS-Branch/          # Security wordlists (placeholder)
```

---

## Port Registry

| Package | Port Range | Notes |
|---------|-----------|-------|
| spectacles | 3700–3799 | FastAPI: 3700 |
| merus-expert | 4300–4399 | FastAPI: 4300 |
| hindsight | 4600–4699 | FastAPI: 4601, Next.js: 4600 |
| mcp-servers | 4900–4999 | Varies by server |
| yevrah_terminal | 5200–5299 | CLI tool, no persistent server |
| agent-swarm | — | Standalone NestJS library (installable via npm) |
| agentic-debugger | — | GitHub Actions only (standalone, config-driven) |
| gbs-integration-validator | 5500–5599 | Fastify: 5510 |
| invoice-reconciliation-tester | 5500–5599 | Fastify: 5520 |
| compliance-auditor | 5500–5599 | Fastify: 5530 |

---

## Environment Variables

See each package's own `.env.example` or `CLAUDE.md` for required environment variables.

---

## Deployment

| Package | Platform | URL |
|---------|----------|-----|
| spectacles | Cloud Run (`glassbox-spectacles`) | `spectacles-gc2qovgs7q-uc.a.run.app` |
| merus-expert | Cloud Run | _(service deployment)_ |
| compliance-auditor | Cloud Run | _(service deployment)_ |
| gbs-integration-validator | Cloud Run | _(service deployment)_ |
| invoice-reconciliation-tester | Cloud Run | _(service deployment)_ |
| mcp-servers | Local / MCP protocol | _(no public URL)_ |
| All others | Local / CI | _(no public URL)_ |

---

## Migration History

| Date | Action |
|------|--------|
| 2026-03-01 | Consolidated 7 standalone repos into this monorepo |
| 2026-03-01 | All source repos archived on GitHub with redirect notices |
| 2026-03-08 | Added 4 agent packages: spectacles, merus-expert, agent-swarm, agentic-debugger |
| 2026-03-08 | Made all 4 agent packages canonical standalone — monorepo is deployment source |
| 2026-03-09 | Added 3 operations audit packages (compliance-auditor, gbs-integration-validator, invoice-reconciliation-tester) |
| 2026-03-09 | Added 2 reference packages (awesome-claude-code, ui-ux-pro-max-skill) |

---

## Documentation Hub Reference

For centralized business, legal, marketing, and product documentation, see the [Adjudica Documentation Hub](~/Desktop/adjudica-documentation/CLAUDE.md) and the [Quick Index](~/Desktop/adjudica-documentation/ADJUDICA_INDEX.md).

For company-wide development standards, see the [Root CLAUDE.md](https://github.com/Glass-Box-Solutions-Inc/adjudica-documentation/blob/main/engineering/ROOT_CLAUDE.md).

---

## ⚠️ GUARDRAILS REMINDER

Before ANY action, verify:

- [ ] **Push permission?** — Required for every push, no exceptions
- [ ] **Definition of Ready?** — Requirements 100% clear
- [ ] **Tests passing?** — 100% required, no workarounds
- [ ] **Root cause understood?** — For fixes, understand WHY first

---

@Developed & Documented by Glass Box Solutions, Inc. using human ingenuity and modern technology

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Glass-Box-Solutions-Inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

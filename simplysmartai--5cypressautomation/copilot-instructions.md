## 5cypressautomation

> > This file mirrors CLAUDE.md and AGENTS.md so the same instructions load in any AI environment.

# Gemini Instructions for 5 Cypress Automation

> This file mirrors CLAUDE.md and AGENTS.md so the same instructions load in any AI environment.
> For the full Operations Bible, see CLAUDE.md. For marketing-specific instructions, see marketing-team/CLAUDE.md.

# 5 Cypress Automation — Operations Bible

**Brand:** 5 Cypress Automation | **Website:** www.5cypress.com | **Contact:** nick@5cypress.com

**Mission:** Deliver two high-value services to small B2B businesses:
1. **Marketing Services** — Strategy, email campaigns, reporting, social content (see `marketing-team/`)
2. **Automation Services** — QuickBooks invoicing, ShipStation, SEO, lead gen, missed-call text-back

**Target clients:** Small B2B tech and medical companies with relationship-driven sales cycles.
**Tech:** Node/TS or Python. APIs: QuickBooks Online, ShipStation, Stripe, Zoho.

---

## Choosing the Right System

| Task | Go to |
|------|-------|
| Strategy brief, email campaign, social content, reporting | `marketing-team/CLAUDE.md` |
| QBO invoice, ShipStation shipping, lead gen, SEO | `directives/` + `execution/` (this file) |
| New client onboarding | `directives/onboard_client.md` |
| Proposal or contract | `directives/create_proposal.md`, `directives/send_contract.md` |

---

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- SOPs written in Markdown, live in `directives/`
- Define goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, update directives with learnings
- You are the glue between intent and execution. Example: read `directives/sales-to-qbo.md`, then run `execution/qbo_invoice.py`

**Layer 3: Execution (Doing the work)**
- Deterministic scripts in `execution/` (Python, with some TypeScript)
- Environment variables and API tokens stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of doing things manually.

**Why this works:** Errors compound. 90% accuracy per step = 59% success over 5 steps. Pushing complexity into deterministic code leaves you focused on decision-making only.

---

## Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per your directive. Only create new scripts if none exist.

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and re-test (check with user first if it uses paid API credits)
- Update the directive with what you learned (API limits, timing, edge cases)

**3. Update directives as you learn**
Directives are living documents. Update them when you discover API constraints, better approaches, or common errors. Do not create or overwrite directives without asking unless explicitly instructed.

---

## Safety Rules (NON-NEGOTIABLE)

- **Financial:** QBO sandbox first, inventory check before invoice
- **Data:** Zod/Pydantic validation on ALL inputs
- **Client:** `.env.example` only, no stored credentials ever
- **Errors:** Log every API call, notify on failure
- **Testing:** Dry-run + 80% coverage before delivery

---

## Self-Annealing Loop

When something breaks:
1. Fix it
2. Update the script/tool
3. Test the fix
4. Update the directive with the new flow
5. System is now stronger

---

## Core Automation Workflows

| Workflow | Directive | Scripts |
|----------|-----------|---------|
| Sales Form → QBO Invoice | `directives/sales-to-qbo.md` | `execution/create_qbo_invoice.py` |
| QBO Invoice → ShipStation | `directives/form_to_invoice_shipping_inventory.md` | `execution/create_shipping_order.py` |
| SEO outreach machine | `directives/seo_sales_machine.md` | `execution/seo_outreach_prepper.py` |
| Lead research | `directives/lead_research_service.md` | `execution/lead_research_orchestrator.py` |
| Missed-call text-back | `directives/workflow_packages.md` | — |
| Monthly insights report | `directives/deliver_monthly_insights.md` | `execution/generate_monthly_insights.py` |
| Client onboarding | `directives/onboard_client.md` | `execution/onboard_client.py` |
| Proposal creation | `directives/create_proposal.md` | `execution/create_proposal.py` |
| Invoice sending | `directives/send_invoice.md` | `execution/create_invoice.py` |

---

## Tech Stack (MANDATORY)

- **Language:** Python/FastAPI (primary), TypeScript/Node.js (secondary)
- **DB:** SQLite/Postgres (dev → client PostgreSQL/MySQL)
- **APIs:** QuickBooks Online OAuth, ShipStation, Stripe, Zoho Calendar
- **Validation:** Pydantic (Python) / Zod (TypeScript) on ALL inputs
- **Hosting:** Client server — Heroku/Vercel/Railway/Docker as needed
- **Testing:** Pytest + dry-run scripts, 80% coverage before delivery
- **Never:** Hard-coded secrets, auto-prod API calls, Google Sheets as the primary DB

---

## Folder Structure

```
marketing-team/       # AI Marketing Team (strategy, email, reporting, content)
├── context/          # Agency profile + per-client context files
├── sops/             # Email, research, reporting SOPs
├── skills/           # 5 marketing skill definitions
├── references/       # Email examples, frameworks, reporting templates
└── output/           # Client deliverables (per-client subfolders)

directives/           # Automation workflow SOPs
execution/            # API scripts (Python/TypeScript)
clients/              # Per-client config + history
config/               # pricing.json, clients.json
documents/            # Templates, contracts, static docs
.tmp/                 # Temp files (gitignored, always regenerated)
.env                  # API keys (never commit)
.env.example          # Client config template (safe to share)
```

---

## Agent Roles

- **Strategist (Claude):** Requirements → flowchart → directive
- **Builder (Execution scripts):** Code + tests per directive
- **Reviewer (Claude):** Validate output, write docs, produce deploy guide

---

## Skills & Automation Agent Knowledge

Custom skills in `.claude/skills/` provide specialized patterns:

| Skill | Use Case |
|-------|----------|
| `backend-development` | API design for QBO/ShipStation integrations |
| `payment-processing` | Stripe/PayPal, webhooks, PCI compliance |
| `python-development` | FastAPI, async patterns, Pydantic |
| `data-validation` | Input validation, spam prevention |
| `customer-sales` | Cold outreach, follow-ups, proposals |
| `code-documentation` | Client runbooks, deploy guides |
| `api-scaffolding` | Spin up REST APIs quickly |

For detailed agent patterns:
- Backend architecture: `agents/plugins/backend-development/agents/backend-architect.md`
- Python expertise: `agents/plugins/python-development/agents/python-pro.md`
- Stripe integration: `agents/plugins/payment-processing/skills/stripe-integration/SKILL.md`

---

## Custom Commands

Available slash commands in `.claude/commands/`:
- `/scaffold-api` — Create new API endpoint with validation
- `/review-code` — Review code for SMB automation best practices
- `/new-directive` — Create workflow directive following 3-layer architecture
- `/outreach-email` — Generate personalized cold outreach email for prospects

---

## Marketing Team Commands

For marketing deliverables, switch context to `marketing-team/CLAUDE.md` and use:
- `/research` — 90-day strategy brief
- `/email` — B2B email campaign
- `/report` — Marketing performance dashboard
- `/content` — LinkedIn content calendar
- `/present` — HTML presentation from deliverables
- `/newclient` — Scaffold client context file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simplysmartai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simplysmartai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

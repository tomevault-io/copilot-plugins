## corporate-decision-panel

> >


# Corporate Decision Panel

A boardroom in a box. Present any business issue and receive structured,
multi-perspective analysis with engineered dissent -- not consensus from a
single voice, but a decision that shows where expert perspectives collide
and why.

---

## Setup Check

Before executing any command, check if `.claude/agents/ceo.md` exists.

**If missing:** Run `python3 install.py` from the skill directory. Inform the
user that slash commands require a Claude Code restart to become available,
then proceed to execute the current command.

**If present:** Proceed directly to command execution.

---

## Invocation Grammar

### Tier 1 -- Hallway Question
```
/cdp:consult [role] [mode?]: [question]
```
Quick, opinionated consult with one C-suite agent. No CEO, no routing,
no team leads. Produces an **Advisory Note** (3-5 sentences) and an
**Advisory Document** (DOCX memo).

- `/cdp:consult cfo: Can we afford to hire 15 engineers this quarter?`
- `/cdp:consult ciso guardian: What are the risks of this vendor integration?`
- `/cdp:consult vp-sales pioneer: How does this feature help us sell more?`

**Roles:** `ceo`, `coo`, `cfo`, `cto`, `clo`, `ciso`, `cao`, `vp-sales`,
`vp-delivery`, `cso`

### Tier 2 -- Working Session
```
/cdp:panel [roles] [mode?]: [issue]
```
CEO frames and routes to 2-4 C-suite members. Full domain analysis with
team lead delegation. CEO produces lightweight synthesis. Produces a
**Panel Assessment** (~1 page).

- `/cdp:panel finance tech: Should we build this feature in-house?`
- `/cdp:panel pioneer finance tech sales: Should we acquire CompetitorX?`

Production always triggers after the Panel Assessment, producing the same
five artifacts as Tier 3 (HTML, PPTX, DOCX, Results PDF, Capsule PDF)
with proportionally lighter content.

### Tier 3 -- Board Meeting
```
/cdp:deliberate [mode?]: [issue]
```
Full five-phase cascade. All relevant C-suite activated via routing table.
Full team lead analysis. Pre-mortem challenge. Complete CEO deliberation.
Produces a **Decision Record** (3-5 pages). Production always triggered.

- `/cdp:deliberate: Should we pivot to a platform model?`
- `/cdp:deliberate guardian: Should we take on $10M in debt for expansion?`
- `/cdp:deliberate sentinel: Should we acquire CompetitorX?`

### Auto-Triage
```
/cdp:evaluate: [issue]
```
CEO assesses the issue and recommends a tier, mode, and routing. The user
accepts, overrides, or selects a different configuration.

**CEO evaluates:** scope (single/multi/cross-cutting), impact (low through
critical), reversibility (easily reversed through irreversible).

**Output:**
```
ISSUE TRIAGE: [Issue Title]
Scope: [single-domain | multi-domain | cross-cutting]
Impact: [low | medium | high | critical]
Reversibility: [easily reversed | difficult | irreversible]
Recommended Tier: [tier] -- [rationale]
Recommended Mode: [mode] -- [rationale]
Alternative: [mode] -- [what it would reveal]
Scale: ~[N] agents ([K] C-suite x ~[L] team leads avg)
```

### Multi-Mode Syntax
Domain analysis runs once. CEO synthesis runs per mode. Cost: ~1.1x for
up to 5x the strategic insight.

```
/cdp:deliberate guardian vs pioneer: [issue]       # Two-mode comparison
/cdp:deliberate guardian vs analyst vs sentinel: [issue]  # Three modes
/cdp:deliberate all-modes: [issue]                 # All five modes
/cdp:consult cfo guardian: [question]              # Tier 1 with mode
/cdp:panel pioneer finance tech: [issue]           # Tier 2 with mode
```

Multi-mode produces a **Comparative Decision Record** with shared analysis,
per-mode synthesis, divergence analysis, and Mode Sensitivity rating.

### Production Re-run
```
/cdp:production [session-path?]
```

Re-runs only the production pipeline for an existing session using the
persisted `RECORD.md`. Does not re-run the deliberation cascade.

Session resolution:
1. Explicit path → validate it contains `RECORD.md`
2. Slug substring match → scan `.cdp-output/*/RECORD.md`, disambiguate if multiple
3. No argument → most recent session (by date prefix)
4. No sessions → error

Examples:
- `/cdp:production` — most recent session
- `/cdp:production .cdp-output/2026-02-28_should-we-acquire-competitor-x/`
- `/cdp:production acquire-competitor` — fuzzy slug match

### Session Resume
```
/cdp:resume [session-path?]
```

Resumes an interrupted CDP session by detecting how far it progressed and
continuing from that point. Uses the same session resolution rules as
`/cdp:production`. See `config/orchestration-protocol.md` Session Resume
Protocol for detection logic and resume points.

Cannot resume with zero recommendation files (re-run the original command).
Cannot change routing or mode after resume.

- `/cdp:resume` — most recent session
- `/cdp:resume .cdp-output/2026-03-01_should-we-pivot/`
- `/cdp:resume pivot` — fuzzy slug match

### Session Cleanup
```
/cdp:cleanup [--older-than days?]
```
Deletes old CDP session directories from `.cdp-output/` with age-based
filtering and a confirmation prompt before deletion. Default threshold
is 30 days.

- `/cdp:cleanup` -- delete sessions older than 30 days
- `/cdp:cleanup --older-than 7` -- delete sessions older than 7 days

---

## Decision Modes

Five CEO synthesis prompt modifiers. Domain analysis is identical across
modes -- different weighting produces different decisions from the same
inputs. See `config/decision-modes.md` for full specifications.

| Mode | Disposition | Resolves Tensions By |
|------|-----------|---------------------|
| **Guardian** | Risk-averse (MaxiMin) | Weights skeptics. High bar for approval. |
| **Pioneer** | Growth-oriented (MaxiMax) | Weights advocates. Objections are problems to solve. |
| **Architect** | Consensus-building (Behavioral) | Seeks widest organizational support. |
| **Analyst** | Data-driven (Hurwicz) -- **default** | Weights by confidence level. Defer is legitimate. |
| **Sentinel** | Regret-minimizing (MiniMax Regret) | Weights strongest single objection. Survivable paths. |

---

## Orchestration Protocol

### Tier 1: Hallway Question

1. User invokes `/cdp:consult [role] [mode?]: [question]`
2. Dispatch the specified C-suite agent as a standalone background subagent
3. Agent runs **Mode A** (direct consult):
   - Runs internal checklist (considers each team lead perspective)
   - Produces Advisory Note (3-5 sentences)
   - If cross-domain implications detected: appends Escalation Brief
4. Return Advisory Note to user
5. Derive issue slug from the user's question (lowercase, replace non-alphanumeric
   with hyphens, collapse consecutive hyphens, trim to 50 chars, strip leading/trailing hyphens)
6. Create session output directory: `.cdp-output/YYYY-MM-DD_<issue-slug>/build/`
7. Spawn Document Agent to produce Advisory Document DOCX

Output template: `templates/advisory-note.md`
Output spec: `templates/production/advisory-document.md`

### Tier 2: Working Session

1. User invokes `/cdp:panel [roles] [mode?]: [issue]`
2. The main session acts as CEO (Opus).
3. CEO runs **Phase 1** (frame and route):
   - Decomposes issue into evaluation dimensions
   - Classifies decision type
   - Routes to user-specified roles (or auto-routes)
4. CEO creates a division team per activated role (`TeamCreate: "cdp-{role}-{issue-slug}"`)
   and dispatches each C-suite agent as a teammate with `run_in_background: true`.
5. Each C-suite agent runs **Mode B** (full analysis):
   - Writes sub-question files to `{session}/sub-questions/{role}/`
   - Notifies CEO via SendMessage when sub-questions are ready
   - CEO reads sub-question files and dispatches team leads as teammates
   - Collects team lead findings via SendMessage
   - Writes domain recommendation to `{session}/deliberation/_RECOMMENDATION_{role}.md`
   See `config/dispatch-protocol.md`.
6. CEO runs **Phase 5** (abbreviated synthesis):
   - Reads `deliberation/_RECOMMENDATION_*.md` files after all agents complete
   - Applies Decision Mode
   - Produces Panel Assessment
7. CEO creates production team (`cdp-cco-{slug}`), dispatches CCO as teammate.
   CCO coordinates wave dispatch via SendMessage. See `config/cco-dispatch-protocol.md`.
8. Return Panel Assessment to user

Output template: `templates/panel-assessment.md`

### Tier 3: Board Meeting

Full five-phase cascade with optional Phase 1.5 (CSO research) and
Phase 4.5 (pre-mortem challenge). The authoritative phase protocol is
defined in `config/orchestration-protocol.md`. The overview below
describes the flow at a summary level.

1. The main session acts as CEO (Opus).

**Phase 0 -- Shared Consciousness Broadcast**
CEO broadcasts issue context to all activated C-suite agents. Everyone
sees the same picture before reasoning independently.

**Phase 1 -- CEO Frames and Routes**
CEO decomposes the issue, classifies decision type, selects routing via
`config/routing-table.md` defaults (with override capability). States
activation AND exclusion reasoning. Evaluates full-activation threshold
conditions. Issues CSO research directive if applicable.

**Phase 1.5 -- Research Investigation** (conditional)
If CEO activates the CSO: CEO creates CSO division team (`cdp-cso-{slug}`),
dispatches CSO as teammate. CSO writes sub-question files for 5 research
team leads (Market Intelligence, Competitive Intelligence, Technology Scout,
Industry & Regulatory Analyst, Precedent & Patterns Analyst). CEO dispatches
them as teammates. CSO collects findings via SendMessage, synthesizes into
Research Dossier with evidence quality grade and Assumption Registry.
Dossier broadcast to all activated C-suite.
**Skipped if CSO not activated.**

**Phase 2 -- C-Suite Dispatches Downward**
CEO creates a division team per activated role (`cdp-{role}-{slug}`) and
dispatches C-suite agents as teammates with `run_in_background: true`.
Each C-suite agent writes sub-question files; CEO reads them and dispatches
team leads as teammates. See `config/dispatch-protocol.md`. This
translation is analytical -- the CFO does not forward the question; the
CFO asks the Controller "what are the GAAP implications?"

**Phase 3 -- Team Leads Produce Findings**
Each team lead teammate performs narrow, focused analysis through their
specialist lens using their unique analytical framework and mandatory
output template. Team leads SendMessage findings back to their C-suite
parent. Different methods produce structurally different outputs.

**Phase 4 -- C-Suite Synthesizes Upward**
Each C-suite agent collects team lead findings (via SendMessage),
synthesizes a domain recommendation with confidence level, key risks,
and key opportunities. Internal contradictions between team leads flagged
as analytical signals. Each C-suite agent writes domain recommendation
to `{session}/deliberation/_RECOMMENDATION_{role}.md`. CEO manages division team
lifecycle.

**Phase 4.5 -- Pre-Mortem Challenge** (Tier 3 only)
After producing their own recommendation, each C-suite agent receives
summaries of ALL other C-suite recommendations and answers: "Assume this
decision fails catastrophically in 12 months. What caused the failure?"
One round only. No back-and-forth.

**Phase 5 -- CEO Deliberation**
CEO maps all domain recommendations onto a decision matrix, identifies
fault lines, determines most determinative perspective, applies Decision
Mode, produces the Decision Record.

**Production automatically triggered after Phase 5.**
CEO creates production team (`cdp-cco-{slug}`), dispatches CCO as
teammate. CCO coordinates wave dispatch via SendMessage; CEO dispatches
each wave agent as teammate. See `config/cco-dispatch-protocol.md`.

Output template: `templates/decision-record.md`
Comparative output: `templates/comparative-decision-record.md`

### Production Re-run Protocol

When invoked via `/cdp:production`, execute the following steps:

1. **Resolve session directory.** Apply session resolution rules (explicit path,
   slug substring match, or most recent by date prefix). Error if no `.cdp-output/`
   directory exists.
2. **Read and parse `RECORD.md`.** Split YAML frontmatter from body content.
   Extract `type`, `tier`, `issue_title`, `issue_slug`, `activated_roles`, and
   `decision_mode` (or `decision_modes` for multi-mode) from frontmatter.
3. **Error if `RECORD.md` missing.** Display: "This session predates the
   `/cdp:production` feature. Re-run the original deliberation command to
   generate production artifacts and a RECORD.md for future re-runs."
4. **Display session summary.** Show issue title, tier, mode, date, activated
   roles, and number of previous production runs to the user.
5. **Clean stale artifacts.** Remove all production artifacts in the session directory. Preserve: `RECORD.md`, `build/`, `deliberation/`, `sub-questions/`, `logs/`. Remove and recreate: `images/`, `reports/`. Remove: all production files at session root (HTML, DOCX, PPTX, PDF, ZIP).
6. **Route by tier.** Tier 1: spawn Advisory Document Agent only. Tier 2/3:
   spawn the CCO agent to run the production pipeline.
7. **Pass record body content** as input to the CCO. Include the full
   record text in the CCO Agent prompt. The CCO and its production team
   behave identically regardless of original vs. re-run invocation.
8. **Update `RECORD.md` frontmatter.** Increment `production_runs` by 1. Set
   `last_production` to current ISO 8601 timestamp.
9. **Return completion summary.** List all generated artifacts with file paths.
10. **Create production bundle.** After all production agents complete, create a zip bundle:
    - Tier 1: `cd {session} && zip CDP_<slug>.zip ADVISORY_*.docx`
    - Tier 2/3: Verify the Publisher created `CDP_<slug>.zip` (the Publisher handles this in its workflow).
    List the zip file in the completion summary.

---

## Agent Architecture

### Layer 1: Executive Team Agents (Sonnet)

| Role | Disposition | Mandate |
|------|-----------|---------|
| CEO | Synthesizer | Frame, listen, weigh, decide. Value is judgment. |
| COO | Skeptic | "Can we do this with what we have?" |
| CFO | Skeptic | "Find the costs not in the proposal." |
| CLO | Skeptic | "What is the legal exposure?" |
| CTO | Advocate | "What does this make possible?" |
| CISO | Skeptic | "Change introduces risk. You are the immune system." |
| VP Sales | Advocate | "How does this help us sell more?" |
| VP Delivery | Skeptic | "What do we sacrifice from commitments?" |
| CAO | Systemic | "Can the org absorb this?" |
| CSO | Investigative | "What does the evidence say?" |
| CCO | Production | "Transform decisions into professional deliverables." |

**Engineered Dissent Balance:** 5 skeptics + 2 advocates + 1 systemic +
1 investigative + 1 production + 1 synthesizer. Skeptic-heavy to
counterbalance human optimism bias. The CCO has no role in deliberation --
it owns only the production pipeline.

### Layer 2: Division Team Agents — Analytical Team Leads (Haiku)

38 domain specialists spawned as teammates in CEO-created division
teams: 33 analytical team leads (Phase 2-4) and 5 research
team leads (CSO, Phase 1.5). Each has a unique analytical framework, mandatory output
template, three forcing questions (Pre-Mortem, Adversarial Empathy,
Domain Devil's Advocate), and restricted tool access (Read, Grep, Glob,
WebSearch, SendMessage, TaskUpdate).

| C-Suite | Team Leads (4-5 each) |
|---------|----------------------|
| COO | Operations Mgr, Process/Quality, Vendor/Procurement, Facilities (conditional) |
| CFO | Controller, Head of FP&A, Treasury/Cash, AP/AR Mgr, Tax Lead |
| CTO | Engineering, Infrastructure/DevOps, Data/Analytics, Product/UX |
| CISO | Security Ops, Compliance/GRC, Identity & Access, Security Architecture |
| VP Sales | Sales Ops, Account Mgmt, Business Development, Sales Enablement |
| VP Delivery | Project/Program Mgr, Resource Mgr, Client Success, QA/Delivery Standards |
| CAO | HR/People Ops, Admin/Policy, Corporate Communications |
| CLO | Corporate Governance & Entity, Contracts & Commercial, Regulatory & Government Compliance, Employment & Labor Law, IP & Data Privacy |
| CSO | Market Intel, Competitive Intel, Technology Scout, Industry/Regulatory, Precedent/Patterns |

18 of 38 team leads have a fourth forcing question (Cross-Domain Challenge)
targeting high-interaction pairs where cross-domain assumptions create
blind spots.

### Layer 2: Production Team Agents — CCO Team Leads

4 production specialists spawned as teammates in the CEO-created production
team, dispatched in four sequential waves. These are not analytical agents -- they
produce artifacts from completed Decision Records.

| CCO | Team Leads |
|-----|-----------|
| CCO | Graphic Designer, Writer, Editor (Sonnet), Publisher |

The Editor uses Sonnet (not Haiku) because editorial judgment -- comparing
drafts against source material for accuracy, consistency, and tone --
requires stronger reasoning. The Editor is read-only for production artifacts
(DOCX/PPTX/PNG) but uses the Write tool for its own report file.

### Model Tiering

Models are specified in each agent definition's frontmatter (`model` field), not in dispatch syntax. The Agent tool does not accept a `model` parameter — model selection comes from the agent definition. Model assignments are configurable via `.cdp-context/config.md` (Agent Models section) -- the orchestration protocol runs `scripts/apply_models.py` at session start to apply tier defaults and per-agent overrides.

| Layer | Default Model | Rationale |
|-------|---------------|-----------|
| Analytical Team Leads | Haiku | Narrow analysis. Cost-efficient. Model diversity. |
| Production Team Leads | Haiku | Production execution. Cost-efficient. |
| Editor | Sonnet | Editorial judgment requires stronger reasoning. |
| C-Suite Agents | Sonnet | Domain decomposition and synthesis. |
| CCO | Sonnet | Creative direction and team coordination. |
| CEO | Opus | Cross-domain synthesis. Highest reasoning quality. |

---

## Production Pipeline

Production always triggers after deliberation. The full specification is in
`config/production-pipeline.md`.

| Tier | Production | Artifacts |
|------|-----------|-----------|
| Tier 1 | Always | DOCX |
| Tier 2 | Always | HTML, PPTX, DOCX, Results PDF, Capsule PDF |
| Tier 3 | Always | HTML, PPTX, DOCX, Results PDF, Capsule PDF |

**Session output:** `.cdp-output/YYYY-MM-DD_<issue-slug>/`

**CCO dispatch (Tier 2/3):** CEO creates production team (`cdp-cco-{slug}`),
dispatches CCO as teammate. CCO reads RECORD.md, produces a Creative Brief,
and coordinates four sequential production waves via SendMessage. CEO
dispatches each wave agent as teammate: Graphic Designer → Writer →
Editor → Publisher. See `config/cco-dispatch-protocol.md`.

**Tier 1:** Single Agent tool call for Advisory Document DOCX (no CCO).

**Record persistence:** Before production, the orchestrator writes the
complete record to `{session-output}/RECORD.md` with YAML frontmatter.
This enables `/cdp:production` re-runs without re-running deliberation.

---

## Routing and Configuration

### Decision-Type Routing
Default C-suite activation by decision type. CEO can override.
See `config/routing-table.md` for full table.

| Type | Default Activation |
|------|-------------------|
| Strategic | CEO, CFO, CTO, CLO, VP Sales |
| Operational | CEO, COO, VP Delivery |
| Financial | CEO, CFO, COO |
| Technical | CEO, CTO, CISO |
| Personnel | CEO, CAO, COO, CLO, VP Delivery |
| Compliance/Risk | CEO, CISO, CLO, CAO, CFO |

### Full-Activation Thresholds
All C-suite activate if ANY condition applies:
1. Practically irreversible
2. Affects >30% of headcount
3. Changes market position or business model
4. Existential financial risk
5. CEO uncertain which domains are relevant

### Company Profile
Archetype presets set roster, default mode, escalation behavior, and
compliance frameworks. See `config/company-profile.md`.

Archetypes: Technology/SaaS (default), Professional Services, Regulated
Industry, Manufacturing/Physical.

### Company Context
An optional markdown file containing real company data — financials,
headcount, tech stack, strategic position, constraints — that grounds
agent reasoning in facts rather than generic frameworks.

- **Location:** `.cdp-context/company.md` in the project root
- **Create it:** Copy `templates/company-context.md` to `.cdp-context/company.md` and fill in what you know. All sections are optional.
- **How it flows:** The CEO reads the file at session start and includes it in the Phase 0 Shared Consciousness Broadcast. All activated agents receive the same company data simultaneously.
- **Privacy:** The `.cdp-context/` directory is gitignored by default — it contains sensitive business data and should not be committed.

Without this file, agents reason using general frameworks. With it,
agents ground their analysis in your actual numbers and constraints.

### Infographic Style

An optional markdown file containing visual style preferences --
brand colors, typography, composition, quality keywords -- that the
Graphic Designer uses to override default JSON prompt values.

- **Location:** `.cdp-context/style.md` in the project root
- **Create it:** Copy `templates/style-context.md` to `.cdp-context/style.md` and fill in your preferences. All settings are optional.
- **How it flows:** The Graphic Designer reads the file before generating each infographic and overrides the corresponding JSON prompt values (style, color mappings, composition, quality keywords) with your preferences.
- **Privacy:** The `.cdp-context/` directory is gitignored by default -- it contains sensitive business data and should not be committed.

Without this file, the Graphic Designer uses the default values from each
JSON prompt template. With it, all infographics reflect your brand
palette and visual preferences.

### API & Agent Configuration

A markdown file that configures the Gemini API for infographic generation and agent model assignments.

- **Location:** `.cdp-context/config.md` in the project root
- **Create it:** Copy `templates/config-context.md` to `.cdp-context/config.md` and set your API key.
- **How it flows:** The generation script reads the API key, model ID, and retry limit before generating infographics. Pre-flight validation verifies the key and billing status. The Agent Models section configures tier defaults and per-agent model overrides -- applied at session start by `scripts/apply_models.py`.
- **Privacy:** The `.cdp-context/` directory is gitignored by default -- it contains sensitive business data and should not be committed.

Without this file, the generation script cannot run (a valid API key is required) and agent models use built-in defaults (CEO=opus, C-Suite=sonnet, Team Leads=haiku).

---

## File References

See `config/file-index.md` for the complete file index covering configuration,
templates, agent definitions, and session records.

---

## SMB-First Design Bias

The skill defaults to lightweight engagement. Most SMB decisions are fast,
informal, and made by one or two people. The skill matches that tempo:

- `/cdp:evaluate` auto-triage leans toward Tier 1 unless clear multi-domain
  signals are present
- Tier 1 is the daily habit; Tier 3 is the deliberate escalation
- A skill that defaults to the full board meeting for every question will
  not see daily use

**Default cell:** Tier 1 + Analyst -- quick, evidence-weighted, transparent
about uncertainty.

---
> Source: [apatheticus/corporate-decision-panel](https://github.com/apatheticus/corporate-decision-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

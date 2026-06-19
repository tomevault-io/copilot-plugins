## pm-skills-claude-code

> **27 skills** organized in **6 domains** covering the full PM lifecycle: from vague idea to scaled product. Includes complete methodological tutorials for the **0→1** journey (idea to PMF) and **1→100** journey (PMF to scale).

# PM Skills for Claude Code — 27 Product Management Skills

**27 skills** organized in **6 domains** covering the full PM lifecycle: from vague idea to scaled product. Includes complete methodological tutorials for the **0→1** journey (idea to PMF) and **1→100** journey (PMF to scale).

Frameworks: Teresa Torres (OST), Ash Maurya (Lean Canvas), Sean Ellis (PMF), Rob Fitzpatrick (Mom Test), Alberto Savoia (Pretotyping), Marty Cagan (Dual-Track), Lenny Rachitsky (Growth + 50 leaders).

---

## Installation

### Option 1: Clone (recommended)
```bash
git clone https://github.com/flowgrammers/pm-skills-claude-code.git
cd pm-skills-claude-code
# Open Claude Code here — all 27 /commands are available
```

### Option 2: Global install (available in any project)
```bash
# Unix/macOS/WSL
./scripts/install-global.sh

# Windows PowerShell
.\scripts\install-global.ps1
```

### Option 3: Manual
Copy files from `.claude/commands/` to `~/.claude/commands/`

---

## Journey 0→1: From Idea to Product-Market Fit

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 1    │    │   PHASE 2    │    │   PHASE 3    │    │   PHASE 4    │
│  Problem     │───▶│  Problem     │───▶│  Solution    │───▶│  Solution    │
│  Discovery   │    │  Validation  │    │  Discovery   │    │  Validation  │
│              │    │              │    │              │    │              │
│ /persona     │    │ /lean-canvas │    │ /opp-tree    │    │ /experiment  │
│ /discovery   │    │ /competitive │    │ /hypothesis  │    │ /ab-test     │
│ /journey     │    │ /hypothesis  │    │ /experiment  │    │ /hypothesis  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
        │                                                           │
        ▼                                                           ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 5    │    │   PHASE 6    │    │   PHASE 7    │
│  MVP Spec    │───▶│  Launch &    │───▶│  Product-    │
│  & Build     │    │  Traction    │    │  Market Fit  │
│              │    │              │    │              │
│ /prd         │    │ /launch-check│    │ /measure-pmf │
│ /user-stories│    │ /gtm         │    │ /interview   │
│ /pre-mortem  │    │ /north-star  │    │ /stakeholder │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Full tutorial**: [tutorial/01-zero-a-um.md](tutorial/01-zero-a-um.md)

---

## Journey 1→100: From PMF to Scale

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 8    │    │   PHASE 9    │    │   PHASE 10   │    │   PHASE 11   │
│  Growth      │───▶│  Scaling &   │───▶│  Optimization│───▶│  Maturity &  │
│  Engine      │    │  Expansion   │    │  & Moat      │    │  Reinvention │
│              │    │              │    │              │    │              │
│ /north-star  │    │ /pricing     │    │ /strategy    │    │ /strategy    │
│ /okr         │    │ /icp         │    │ /competitive │    │ /discovery   │
│ /experiment  │    │ /battlecard  │    │ /okr         │    │ /lean-canvas │
│ /ab-test     │    │ /roadmap     │    │ /stakeholder │    │ /prioritize  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

**Full tutorial**: [tutorial/02-um-a-cem.md](tutorial/02-um-a-cem.md)

---

## 27 Skills by Domain

### DISCOVERY (7 skills) — Understand the problem and market

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/persona` | Generates personas with JTBD, behaviors, pain points | Project start, before any spec |
| `/discovery` | Structures discovery cycle (framing → hypotheses → experiments) | Ill-defined problem, multiple hypotheses |
| `/interview-synthesis` | Synthesizes interviews into JTBD, patterns, recommendations | After interview rounds (3+) |
| `/competitive-analysis` | Maps strengths, weaknesses, differentiation gaps | Preparing PRD or strategy |
| `/opportunity-tree` | Teresa Torres OST (outcome → opportunities → solutions) | Deciding where to invest in discovery |
| `/hypothesis` | Structures testable hypotheses with metrics and kill criteria | Before any experiment |
| `/customer-journey` | Journey map (7 stages + emotions + pain points) | Understanding end-to-end experience |

### DELIVERY (3 skills) — Specify the solution

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/prd` | Complete PRD in 8 sections | Validated hypothesis, time to spec |
| `/user-stories` | INVEST user stories with acceptance criteria | PRD approved, break down for eng |
| `/acceptance-criteria` | Given/When/Then covering happy path + edge cases | Story approved, detail for QA |

### STRATEGY (7 skills) — Prioritize, plan, align

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/prioritize` | RICE scoring + trade-offs + recommendation | 5+ features, need to sequence |
| `/strategy` | Strategy canvas (9 sections) | New quarter, direction change |
| `/roadmap` | Quarterly roadmap with phases and dependencies | Communicate plan to team/board |
| `/okr` | Cascaded OKRs (company → team) | Quarter start, align goals |
| `/lean-canvas` | Ash Maurya canvas (9 blocks) | Quick business model validation |
| `/pricing` | 7 pricing models + willingness-to-pay | Define/revise monetization |
| `/north-star` | NSM + 3-5 input metrics by business type | Define main product metric |

### VALIDATION (3 skills) — Test and measure

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/experiment-design` | Pretotype and A/B test design (XYZ hypothesis) | Before building anything |
| `/measure-pmf` | Evaluates Product-Market Fit (Sean Ellis + retention) | Product with 50+ active users |
| `/ab-test-analysis` | Statistical analysis of A/B test results | After running experiment |

### EXECUTION (2 skills) — Prepare and protect the launch

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/pre-mortem` | Risk analysis (Tigers/Paper Tigers/Elephants) | 2-3 weeks before launch |
| `/launch-checklist` | Cross-functional checklist (eng, design, mkt, sales, support) | 1-2 weeks before launch |

### COMMUNICATION & GTM (5 skills) — Communicate and go to market

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/release-notes` | Benefit-oriented release notes | Feature shipped from dev |
| `/stakeholder-update` | Executive status (situation → analysis → recommendation) | Sprint end, board meeting |
| `/gtm` | Go-to-market (channels, messaging, metrics, timeline) | Feature/product launch |
| `/battlecard` | Competitive battlecard for sales | Position against competitor |
| `/ideal-customer-profile` | ICP with demographics, behaviors, JTBD, needs | Define who to pursue (GTM) |

---

## Ready-Made Workflows (Skill Combos)

| Workflow | Skills | When to use |
|----------|--------|-------------|
| **Customer Discovery** | `/persona` → `/discovery` → `/interview-synthesis` → `/opportunity-tree` | Starting from scratch |
| **Feature Kickoff** | `/hypothesis` → `/prd` → `/user-stories` → `/acceptance-criteria` | Specify a feature |
| **Product Strategy** | `/competitive-analysis` → `/strategy` → `/okr` → `/roadmap` → `/stakeholder-update` | Plan the quarter |
| **Validate & Launch** | `/experiment-design` → `/pre-mortem` → `/launch-checklist` → `/gtm` | Test and launch |
| **Pricing & GTM** | `/pricing` → `/ideal-customer-profile` → `/battlecard` → `/gtm` | Monetize and position |
| **Growth Sprint** | `/north-star` → `/experiment-design` → `/ab-test-analysis` → `/okr` | Accelerate growth |
| **Board Update** | `/measure-pmf` → `/stakeholder-update` → `/roadmap` | Investor meeting |

**Full details**: [tutorial/03-workflows.md](tutorial/03-workflows.md)

---

## How to Use

### 1. Call a skill
```
/discovery
Problem: High onboarding churn (35% in 7 days)
Context: B2B SaaS CRM, 400 users, mid-market
Constraints: 2 weeks to decide whether to write a PRD
```

### 2. Chain skills
```
/persona → output: Persona "Alex, Sales Rep"
     ↓
/discovery → input: Persona Alex + churn problem
     ↓
/prd → input: Validated hypothesis from discovery
     ↓
/user-stories → input: Approved PRD
```

### 3. Refine output
```
/prd [previous context]
Refine: Add privacy section with GDPR compliance
```

### Golden rules
- **Minimum context**: Problem + audience + constraints
- **The more specific, the better**: Data > opinions
- **Skills are independent**: But much more powerful when chained
- **Iterate**: First draft → refine → validate

---

## Tutorials

| File | Content |
|------|---------|
| [tutorial/00-como-usar.md](tutorial/00-como-usar.md) | Quick start, skill map, rules |
| [tutorial/01-zero-a-um.md](tutorial/01-zero-a-um.md) | Journey 0→1: 7 phases from idea to PMF |
| [tutorial/02-um-a-cem.md](tutorial/02-um-a-cem.md) | Journey 1→100: 4 phases from PMF to scale |
| [tutorial/03-workflows.md](tutorial/03-workflows.md) | 7 ready-made workflows with examples |

---

## Directory Structure

```
pm-skills-claude-code/
├── CLAUDE.md                          ← You are here (central hub)
├── README.md                          ← Installation & catalog
├── .claude/commands/                  ← 27 invocable skills
│   ├── persona.md
│   ├── discovery.md
│   ├── interview-synthesis.md
│   ├── competitive-analysis.md
│   ├── opportunity-tree.md
│   ├── hypothesis.md
│   ├── customer-journey.md
│   ├── prd.md
│   ├── user-stories.md
│   ├── acceptance-criteria.md
│   ├── prioritize.md
│   ├── strategy.md
│   ├── roadmap.md
│   ├── okr.md
│   ├── lean-canvas.md
│   ├── pricing.md
│   ├── north-star.md
│   ├── experiment-design.md
│   ├── measure-pmf.md
│   ├── ab-test-analysis.md
│   ├── pre-mortem.md
│   ├── launch-checklist.md
│   ├── release-notes.md
│   ├── stakeholder-update.md
│   ├── gtm.md
│   ├── battlecard.md
│   └── ideal-customer-profile.md
├── tutorial/                          ← Methodology guides
│   ├── 00-como-usar.md
│   ├── 01-zero-a-um.md
│   ├── 02-um-a-cem.md
│   └── 03-workflows.md
├── templates/                         ← Fillable templates
│   ├── prd-template.md
│   ├── user-story-template.md
│   ├── discovery-template.md
│   └── strategy-canvas.md
└── scripts/                           ← Global installation
    ├── install-global.sh
    └── install-global.ps1
```

---

## Frameworks & Sources

| Framework | Author | Used in |
|-----------|--------|---------|
| Opportunity Solution Tree | Teresa Torres | `/opportunity-tree` |
| Lean Canvas | Ash Maurya | `/lean-canvas` |
| Mom Test | Rob Fitzpatrick | `/discovery`, `/interview-synthesis` |
| PMF Survey (40%) | Sean Ellis | `/measure-pmf` |
| Superhuman PMF Engine | Rahul Vohra | `/measure-pmf` |
| Pretotyping / YODA | Alberto Savoia | `/experiment-design` |
| Growth Loops | Reforge | `/north-star` |
| RICE Scoring | Intercom | `/prioritize` |
| INVEST Stories | Mike Cohn | `/user-stories` |
| Gherkin/BDD | Cucumber | `/acceptance-criteria` |
| OKR Principles | 55 Product Leaders (Lenny) | `/okr` |
| Pre-mortem | Gary Klein | `/pre-mortem` |
| Value-Based Pricing | Paddle/Price Intelligently | `/pricing` |
| ICP Framework | Multiple | `/ideal-customer-profile` |
| AARRR Pirate Metrics | Dave McClure | Tutorial 0→1 |
| Three Horizons | McKinsey | Tutorial 1→100 |

---

## Conventions

### Input
- **Minimum**: Problem + audience + constraints
- **Format**: Markdown with bullets or sections
- **Data**: The more real data, the better the output

### Output
- **Structured**: Always in numbered, clear sections
- **Markdown**: Formatted to copy directly (Notion, Confluence, Google Docs)
- **Actionable**: Focus on explicit next actions

### Language
- Descriptions in **Portuguese (pt-BR)** by default
- Technical terms and frameworks in **English**
- Outputs adapt to the language of your context

---
> Source: [lucasgaravelli/pm-skills-claude-code](https://github.com/lucasgaravelli/pm-skills-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->

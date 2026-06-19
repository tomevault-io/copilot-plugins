## skill-builder-guide-skill

> >-


> *授人以鱼，不如以渔。* ——《淮南子·说林训》
> 技能教会智能体怎么做事，这份文件教会技能怎么诞生。

# Skill Builder — AI Agent Skills Creation Guide

> **Position**: meta tier skill. Output = new skill files (SKILL.md + openai.yaml), not business code.
>
> **Deployment**: `cp -r SKILL-BUILDER-GUIDE/ target-project/.claude/skills/skill-builder/` — then auto-loaded as `/skill-builder`.
>
> **Progressive loading**: This file is L2. Deep methodology in `references/` — loaded only when a phase or scenario triggers them (§2.7).

## Trigger Conditions

- Creating a skill system for a project
- "create skill" / "generate project-specific skill" / "skill template"
- "model tier" / "L0 delegation" / "skill validation"
- "openai.yaml" / "frontmatter spec"
- "how to decompose" / "delegation pattern"

---

## 1. Core Concepts

### 1.1 Dual-Axis Model

Every skill is defined on **two independent, orthogonal dimensions**:

| | Execution (model_tier) | Composition (skill_tier) |
|--|------|------|
| Question | Who executes? | Where in the dependency graph? |
| Values | L0 / L1 / L2 / L3 | meta / planning / functional / atomic |

**Execution**: L0=Haiku (mechanical) · L1=Sonnet (bounded) · L2=Sonnet/Opus (multi-step) · L3=Opus (architectural)

**Composition**: meta=creates skills · planning=task routing · functional=multi-step routines · atomic=single source

**Consistency**: L0+meta invalid · L0+planning invalid · L0+functional not recommended · L0+atomic valid

### 1.2 Model Tier Routing

| Tier | Cognitive Load | Model | Typical Tasks |
|:----:|---------|---------|---------|
| **L0** | Mechanical | Haiku | File lookup, info query, command exec, static tracing |
| **L1** | Bounded | Sonnet | Single-module changes, narrow search, bounded implementation |
| **L2** | Reasoning | Sonnet/Opus | Root cause diagnosis, cross-module implementation |
| **L3** | Strategic | Opus | Architectural decisions, security audits |

**Mandatory rule**: L0 tasks → Haiku. Main model executing L0 = 5-15x token waste.

| L0 Task | Example | Delegate To |
|---------|------|---------|
| File lookup | "Where is the entry file" | Haiku |
| Info query | "What framework version" | Haiku |
| Command exec | "Run deploy script" | Haiku |
| Mechanical edit | "Update version number" | Haiku |

**Upgrade triggers**: ≥2 correction rounds failed → upgrade model. Same code ≥3 modifications → stop, re-analyze. 3 rounds without narrowing → upgrade to re-diagnose.

### 1.3 Skill Type Catalog

| Skill Type | Execution | Composition | Required Sections | Handoff | Template |
|---------|:--:|:--:|------|---------|------|
| Standards (dev) | L1 | atomic | Tech stack, layered arch, code standards | → code-map (new files ≥3) | `templates/example-dev/` |
| Code Map (code-map) | **L0** | atomic | Dir structure, quick lookup, routes | → dev (conventions lookup) | `templates/example-code-map/` |
| Workflow (workflow) | L1 | functional | Phase→input→steps→output, checklists | → change-model (changes ≥2 modules) | `templates/example-workflow/` |
| Change Model (change-model) | L1 | functional | WHY/WHAT/HOW/VALIDATION, call-chain check. **Implicit trigger**: at CONFIRM phase, Agent asks whether to generate a change report for the completed work. | → call-chain (changes ≥3 layers) | `references/change-model.md` |
| Call-Chain (call-chain) | L1 | functional | Tracing method, type checking, final-call checklist | → change-model (new change found) | `templates/example-call-chain/` |
| Scripts (scripts) | L0 | atomic | Script inventory, execution flow, error handling. **Min threshold**: ≥5 standalone scripts or ≥1 complex deployment pipeline. | — | — |
| Delegation (delegation) | L1 | planning | Decomposition criteria, model routing | → any (as needed) | `references/delegation.md` |

### 1.4 Skill Selection — Decision Guide

More skills ≠ better. During initialization, the agent guides the human through three questions. The answers naturally lead to a skill set. No preset combos.

**Three Decision Questions**:

| # | Question | Options |
|---|------|------|
| 1 | What's your project scope? | Quick fixes / solo prototype · Ongoing product / small team · Multi-module / multi-service |
| 2 | How autonomous should the AI be? | Passive lookup only · Independent within a module · Cross-module analysis, proactive reporting |
| 3 | Do you have existing conventions documented? | No, extract from code · Yes, use them |

**How answers map to skills**:

```
dev — every project needs it
code-map — recommended when project scope > "solo prototype"
workflow — recommended when AI autonomy > "passive"
change-model — recommended when AI autonomy = "cross-module"
call-chain — multi-service + cross-module autonomy
delegation — team >1 or cross-module autonomy
```

**Quick-start or full pipeline**: The human can always say "quick start" (skip deep scan and validation, ≤5 min) or "full pipeline". The agent doesn't presume which is better.

Full methodology: `references/decision-guide.md`.

### 1.5 Skill Chains

Skills shouldn't trigger in isolation. After completing one skill, recommend the next based on output characteristics.

| Entry Skill | After Completion, Recommend | Trigger Condition |
|---------|-----------|---------|
| dev | `{project}-code-map` | new files ≥3 |
| dev | `{project}-workflow` | non-standard directory structure found |
| code-map | `{project}-dev` | need to look up conventions |
| workflow | `{project}-change-model` | changes span ≥2 modules |
| change-model | `{project}-call-chain` | changes span ≥3 layers (API→Service→Data→DB) |
| call-chain | `{project}-change-model` | new change requirement discovered |
| delegation | (any as needed) | based on decomposition result |

**Implementation**: Each generated SKILL.md includes a `## Handoff` section at the end (see §2.4 hard rule 11). Full methodology: `references/skill-chain.md`.

---

## 2. Execution Pipeline

The pipeline operates on an **Execution Tree** — the unified model that combines task decomposition (H-ADMC), execution assignment (delegation), and context loading (progressive disclosure). Each pipeline phase maps to a tree lifecycle stage: ANALYZE builds the root → SCAN fills the leaves → GENERATE produces output nodes → VALIDATE verifies → CONFIRM closes. Full specification: `references/execution-tree.md`.

### 2.1 Pipeline Overview

完整五阶段管线（全部走或选着走，人类决定）：

```
ANALYZE (L1) → SCAN (L0) → GENERATE (L1) → VALIDATE (L0) → CONFIRM (L1)
    │              │              │                │               │
  Skill plan   Code samples   SKILL.md        Validation     approved/revise
  + CLAUDE.md                 + openai.yaml
  modules                     + CLAUDE.md
```

| Phase | Executor | Input | Output | Gate |
|------|:------:|------|------|------|
| ANALYZE | Sonnet (L1) | Project overview | Skill plan: `[{name, tier, reason}]` | **Pause for user confirmation** |
| SCAN | **Haiku (L0)** | Skill plan → codebase | Code samples per layer | All layers scanned |
| GENERATE | Sonnet (L1) | Scan results | SKILL.md + openai.yaml + CLAUDE.md + references/ | 10 hard rules met |
| VALIDATE | **Haiku (L0)** | Generated files | Pass/Fail report | V1+V2 passed |
| CONFIRM | Sonnet (L1) | Validated skills | approved / revise / reject | User explicit response |

**Quick-start**: When the human says "quick start" / "先跑起来", skip SCAN→VALIDATE. Grab key data, fill templates, deliver in ≤5 min. Prompt to optionally re-run VALIDATE afterward. See `references/pipeline/phase-3-generate.md`.

**Decision guide**: During initialization, the agent does not decide which skills to generate on the human's behalf. Ask three questions; let the answers naturally surface the right set. See `references/decision-guide.md`.

### 2.2 ANALYZE — Project Analysis

- **Executor**: Sonnet (L1). Analyze project type, tech stack, module structure, team size.
- **Output**: Skill plan in `[{name, skill_tier, model_tier, reason}]` format + CLAUDE.md module selection `{modules: ["M1", ...]}` per project profile (see `references/claude-md-spec.md` §4).
- **Gate**: Pause for user confirmation. Do NOT proceed to SCAN without approval.
- **Reference**: `references/pipeline/phase-0-example.md` — granularity benchmark (NestJS).

### 2.3 SCAN — Code Scanning

- **Executor**: Haiku (L0) — **must delegate**. Main model dispatches Haiku sub-agents; does NOT execute scan directly.
- **Output**: Code samples per layer with pattern identification.

| Layer | Scan Target | Record |
|------|---------|--------|
| API Layer | Route declarations, param validation, response wrapping | `{method}`, `{class/function name}` |
| Service Layer | Abstract interfaces, consistency management, param conversion | `{yes/no}`, `{method}` |
| Data Layer | ORM approach, query organization, pagination | `{framework}`, `{method}` |
| Integration Layer | Remote calls, message queues, scheduled tasks | `{list}`, `{config}` |
| Existing Conventions | Project's existing README, CHANGELOG, CONTRIBUTING, docs/, change logs | Document paths, format, and naming — generated skills must incorporate or explicitly replace these |
| Reusable Workflows | Repeated operations: build, deploy, test suites, data migration, code generation | Shell scripts, npm scripts, Makefile targets — evaluate for scripts/ skill or workflow automation |

- **Reference**: `references/pipeline/phase-2-scan.md` — project type → layer mapping.
- **Gate**: ☐ All scan sub-tasks dispatched to Haiku? ☐ Existing project conventions/documents detected? ☐ Reusable scripts/workflows identified?

### 2.4 GENERATE — Create Skill Files

- **Executor**: Sonnet (L1).
- **Output**: SKILL.md + openai.yaml + CLAUDE.md → `.claude/skills/{name}/` (production) or `skills/{name}/` (template).

**Hard rules**:
1. Frontmatter: name, description, model_tier, skill_tier, version, status (`references/frontmatter-spec.md`)
2. Trigger words from project code: 5-15, action + query types
3. Actual version numbers from scan — no placeholders
4. Scanned code snippets, sanitized (IPs/passwords → `{placeholder}`)
5. Cross-reference related skills with relative paths
6. SKILL.md body ≤5000 tokens
7. Progressive loading: functional-tier skills (workflow, change-model, call-chain) MUST create `references/` for content exceeding the L2 body budget. If the skill body approaches 5000t, split detailed templates/methodologies into `references/`. Atomic-tier skills (dev, code-map, scripts) may omit references/ if the lookup table fits in the body.
8. Load methodology for the skill type: when generating a skill of type X, load the corresponding L3 reference first (`references/change-model.md` for change-model skills, `references/execution-tree.md` for delegation skills). The generated skill must reflect the current naming conventions and patterns from its reference.
9. Completeness checklist: before delivering generated skills, verify all items in §2.4.1.
10. CLAUDE.md generation: load `references/claude-md-spec.md`. Fixed skeleton (A-D) always present. Optional modules (M1-M6) selected during ANALYZE per project profile (§2.2). Data sourced from SCAN results, not re-collected.
11. **Handoff section**: Each generated SKILL.md must include a `## Handoff` section at the end, declaring which skills to recommend after completion and under what conditions. Recommendation source: §1.5 Skill Chains table. Full methodology: `references/skill-chain.md`.

**Language**: Concise English. Trigger words match project language.
**Reference**: `references/pipeline/phase-3-generate.md` — per-skill-type generation prompts.

**Naming disambiguation**: Risk levels in change-model skills use R0-R3 (not L0-L3, which are execution tiers). See `references/change-model.md` §3.2.

**Gate**: ☐ All 11 hard rules passed? ☐ Completeness checklist (§2.4.1) all ✅? ☐ CLAUDE.md generated with correct module set?

#### 2.4.1 Completeness Checklist

Before delivering generated skills, verify every item:

- [ ] **Progressive loading**: functional-tier skills (workflow, change-model, call-chain) have `references/` directory if body exceeds 3000 tokens
- [ ] **Model delegation**: L0 skills body states "must delegate to Haiku"; L1 skills note which sub-tasks delegate to L0
- [ ] **Naming disambiguation**: risk levels use R0-R3 (not L0-L3); execution tiers use L0-L3
- [ ] **Existing conventions**: project's existing docs, changelogs, workflows are either incorporated or explicitly replaced (not silently ignored)
- [ ] **Cross-references**: all relative paths resolve to existing files within the target project
- [ ] **Technical accuracy**: version numbers, class names, file paths, and dependency labels verified against target project source (not assumed or approximated)
- [ ] **Trigger words**: 5-15 per skill, project-specific (no generic terms like "develop" or "modify")
- [ ] **No placeholders**: every `{placeholder}` replaced with actual content from scan
- [ ] **Token budget**: each SKILL.md ≤5000 tokens; CLAUDE.md ≤1500 tokens
- [ ] **CLAUDE.md modules**: selected during ANALYZE, data sourced from SCAN (not re-collected)
- [ ] **CLAUDE.md quick commands**: each command copy-paste executable, no placeholders
- [ ] **CLAUDE.md skill routing**: matches generated skills one-to-one
- [ ] **Handoff section**: Each generated SKILL.md includes `## Handoff`, recommendations match §1.5 Skill Chains, referenced skills exist in the target project

### 2.5 VALIDATE — Verify

- **Executor**: Haiku (L0) — **must delegate**.

```bash
python scripts/validate-skills.py .claude/skills/{name}
python scripts/validate-skills.py skills/{name}
```

| Layer | Pass Condition |
|--------|---------|
| V1 Format | Frontmatter complete, triggers ≥5, YAML valid, no forbidden fields |
| V2 Structure | Dual-axis consistent, skill references exist, relative paths correct |
| V3 Semantic | File paths ≥95%, method names ≥90%, version numbers 100% — run with `--semantic` |
| V4 CLAUDE.md | Quick commands executable, skill routing consistent, tech stack versions match SCAN, zero placeholders (see `references/claude-md-spec.md` §6) |

**Acceptance criteria**:

| Declaration | Verification | Pass Rate |
|---------|---------|:---:|
| File paths | Glob/Read confirm existence | ≥95% |
| Method names | Grep source confirm | ≥90% |
| Version numbers | Read dependency config | 100% |

Below standard → must not publish. Verify with tools, not by trusting documents.
**Reference**: `references/validation-protocol.md` — full protocol.
**Gate**: ☐ V1+V2 passed? ☐ V3 semantic passed (if available)? ☐ V4 CLAUDE.md passed? ☐ Zero broken cross-references?

### 2.6 CONFIRM — User Approval

- **Executor**: Sonnet (L1).
- **Output**: `approved` | `revise({feedback})` | `reject`.

Confirm: version numbers match scan, standard patterns correct, user docs merged (user docs take precedence over scan). On revise: apply feedback, re-submit (max 2 rounds, then escalate).

**Gate**: ☐ User explicit response received? ☐ If this pipeline execution modified the project, change report offered to user? (Agent asks: "Generate a change report for this session?" — see `references/change-model.md`) ☐ Reusable workflows identified during SCAN offered for script/template archival?

### 2.7 L3 Routing Table

Load deeper methodology only when the corresponding phase or scenario triggers it:

| Scenario / Phase | Load | Content |
|------|------|------|
| Initialization decision | `references/decision-guide.md` | Three-question decision tree, natural skill selection |
| Phase 1 ANALYZE | `references/pipeline/phase-0-example.md` | NestJS end-to-end example |
| Phase 2 SCAN | `references/pipeline/phase-2-scan.md` | Project type → layer mapping |
| Phase 3 GENERATE (skills) | `references/pipeline/phase-3-generate.md` | Per-skill-type generation prompts, quick-start path |
| Phase 3 GENERATE (CLAUDE.md) | `references/claude-md-spec.md` | Modular architecture, behavioral constitution, module selection matrix |
| Phase 4 VALIDATE | `references/validation-protocol.md` | V1/V2/V3 specs, pass rates |
| Filling frontmatter | `references/frontmatter-spec.md` | Required/forbidden/removed fields |
| Task decomposition, execution tree, delegation patterns | `references/execution-tree.md` | H-ADMC criteria, AND/OR/LEAF nodes, patterns A/B/C, C/B/U format |
| Change reports, archiving | `references/change-model.md` | 4-layer architecture, 8-item checklist, R0-R3 risk levels |
| Skill chains, handoff | `references/skill-chain.md` | Handoff 段格式, 3 种 Chain Pattern, 激活率指标 |
| Global skills awareness | `references/global-skills-awareness.md` | Embedding global skills reminder in generated output |

### 2.8 Deep Review (Optional Phase 6)

After CONFIRM, the user may request an independent quality evaluation of generated skills. This spawns a fresh agent in worktree isolation that evaluates: practical value, over-design, logical conflicts, development guidance, engineering stability. See `docs/evaluation-report-*.md` for evaluation rubric.

Trigger: user says "review the generated skills" / "evaluate the skill system" / "深度复核".

---

## 3. Deployment

### 3.1 Directory Structure

```
{skill-name}/
├── SKILL.md              # L2: Core body (≤5000 tokens)
├── agents/
│   └── openai.yaml       # L1: Trigger config (5-15 triggers)
├── references/           # L3: Deep reference, on demand
├── scripts/              # L4: Executables (zero context)
└── assets/               # Static resources
```

| Path | Use Case | Auto-Load |
|------|---------|:---:|
| `.claude/skills/{name}/` | Production project skills | ✅ |
| `skills/{name}/` | Template/methodology projects | ❌ |
| `~/.claude/skills/{name}/` | Global cross-project skills | ✅ |

Naming: `{project}-{type}`, lowercase, hyphen-separated. Dir name = skill name.

### 3.2 CLAUDE.md — Modular Architecture

CLAUDE.md is a **formal Phase 3 output**, generated alongside SKILL.md + openai.yaml. It serves two roles: (1) shape Agent behavior toward the user, (2) cache high-frequency project data to avoid repeated L0 lookups.

**Architecture**: fixed skeleton + optional modules, selected during ANALYZE per project profile.

| Section | Type | Content |
|------|------|------|
| A. Project Identity | Fixed | 1-line description |
| B. Behavioral Constitution | Fixed | 7 rules — 4 Karpathy principles (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) + 3 project principles (Discover→Report, Change→Test, Complete→Archive) |
| C. Skill Routing | Fixed | Auto-loaded skill index |
| D. Quick Commands | Fixed | build / run / test (from SCAN) |
| M1. Tech Stack | Default on | Versions from dependency config |
| M2. Key Directories | Optional | 5-10 dirs, ≥3 modules trigger |
| M3. Architecture Overview | Optional | Layer/data flow, complex projects trigger |
| M4. Coding Constraints | Optional | Non-obvious rules, non-standard patterns trigger |
| M5. Domain Glossary | Default on | All projects; 0-5 terms → "basic, needs supplement"; ≥5 → "core" |
| M6. External Dependencies | Optional | ≥3 external services trigger |

**Behavioral Constitution** (Section B) encodes 7 behavioral rules: 4 from Karpathy (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution) and 3 project principles (Discover→Report, Change→Test, Complete→Archive). These are cross-skill default behaviors, orthogonal to skill-specific instructions.

Full specification: `references/claude-md-spec.md` — generation rules, data source mapping, validation checklist.

Placement: `.claude/skills/` projects use `CLAUDE.md` at project root (auto-loaded). `skills/` projects embed the skill routing table within the generated SKILL.md instead.

### 3.3 Conflict Resolution

When multiple skill triggers overlap:

| Priority | Rule | Example |
|:--:|------|------|
| 1 | Project-specific > generic | `myproject-dev` over `example-dev` |
| 2 | Required > recommended | Explicit directive over suggestion |
| 3 | Exact match > broad match | "file location" → code-map, not dev |
| 4 | Active > deprecated | Skip superseded/deprecated skills |

### 3.4 Global Skills Awareness

After initializing project-specific skills, the agent may overlook globally installed skills (`~/.claude/skills/`). Embed this block at the end of generated CLAUDE.md §C (≤100 words):

```markdown
### Global Skills

Project skills take priority, but don't forget globally installed skills (`~/.claude/skills/`).
Use `/skills` to list all available skills. Don't reimplement what a global skill already does.
```

Also append a one-line hint to the SKILL.md template's `## Related Skills` section. See `references/global-skills-awareness.md`.

---

## 4. Reference

### 4.1 Templates Index

| Template | Path |
|------|------|
| Standards (dev) | `templates/example-dev/` |
| Code Map (code-map) | `templates/example-code-map/` |
| Workflow (workflow) | `templates/example-workflow/` |
| Call-Chain (call-chain) | `templates/example-call-chain/` |
| Delegation (delegation) | `templates/example-delegation/` |
| SKILL.md skeleton | `templates/skill-template.md` |
| openai.yaml skeleton | `templates/openai-template.yaml` |
| Change report | `templates/change-model-template.md` |
| 小本本 (emotion log) | `templates/example-xiaobenben/` |

Each template is a complete mini-skill (SKILL.md + openai.yaml + references/). Copy to `.claude/skills/` and customize.

### 4.2 Import Steps

```
① Analyze project (type, tech stack, team size) → ② Decision guide (§1.4): ask three questions, let answers select skills → ③ Generate to `.claude/skills/` → ④ Validate + iterate
```

### 4.3 Conventions

**Versioning**: SemVer. Commit format: `feat|fix|docs(skills): {description}`.

**Security**: Sanitize code examples (secrets → `{placeholder}`). No reference skill may contain real project secrets. No agent signatures or promotional links in generated files.

**Maintenance**: Update skills on tech stack change, new module, workflow optimization, systematic errors. Validate after each change.

**Lifecycle**: `draft → active → deprecated → (removed)` or `superseded → replacement`.

---
> Source: [ang-XWBWZ/SKILL-BUILDER-GUIDE-SKILL](https://github.com/ang-XWBWZ/SKILL-BUILDER-GUIDE-SKILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

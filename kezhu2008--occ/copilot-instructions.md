## occ

> Use when standing up, running, or re-staffing an AI "company" that builds and maintains a specific code repository — turning a founder's business intent into a structured org of CEO / manager / executor / reviewer / HR persona skills scoped to that repo. Triggers include "spin up a team for this project", "run my company", "review the org", or wanting autonomous repo development with minimal founder intervention.


# OCC — One Consultancy Company

## Overview

OCC is the **staffing-and-consulting firm**. It does not build your product. It builds (and re-builds) the **org that builds your product**: a structured team of persona skills scoped to one code repository.

**A company = one code repository pursuing one business intent.**

OCC has exactly two capabilities — the same two a real consultancy sells:

1. **Strategic review** (`occ-strategic-review`) — interview the founder, design or revise the org: roles, reporting lines, job descriptions. Output is versioned org design in `company_policies/`.
2. **Recruiting** (`occ-recruiting`) — instantiate that design into real, invokable persona skills under the repo's `.claude/skills/`, each reusing superpowers and each paired with a counter-reviewer.

OCC makes **structural** change (new/removed/redefined roles). Inside the company, **HR** makes **incremental** change (training a persona from accumulated feedback). Two different gears — keep them separate. See `references/corporate-structure.md`.

## When to use

- "Set up a team to build/maintain `<repo>`" → run `occ-strategic-review`, then `occ-recruiting`.
- "The team is missing a role / a role is wrong" → `occ-strategic-review` (revise), then `occ-recruiting` (re-recruit just the delta).
- "Run my company / ship the next milestone" → you are past OCC; invoke the recruited **CEO** persona (`<company>-ceo`). OCC built it; the CEO runs it.
- Coaching one persona to behave better without changing the org → that's **HR** (`<company>-hr`), not OCC.

## The three-tier storage model (read this first)

Everything OCC and the company produce lands in one of three tiers. Confusing them is the most common failure.

| Tier | Lives in | Lifespan | Examples |
|------|----------|----------|----------|
| **Persistence** (authoritative facts) | `<repo>/company_policies/` | Whole project, git-tracked | charter, org-chart, roadmap, architecture, data-model, **specs (R#/A#)**, use-cases, tasks, decisions, performance |
| **Knowledge** (shared map + gotchas) | `<repo>/company_policies/knowledge/` | Until superseded | "FX routes through Div775Engine not CGT — see `Div775Engine.swift:40`; missing a USD movement silently under-reports tax" |
| **Memory** (short-term role feedback) | `<persona>/MEMORY.md` | Until HR triages | "CEO over-explained architecture", "iOS dev skipped device check" |

Knowledge is **shared** and kept small by construction — entries *reference* the authoritative artifact, never restate it; HR gardens it (dedup, supersede, budget). **HR** is the only function that promotes: behavior → SKILL.md (writing-skills, RED baseline), gotcha → knowledge/, fact → a spec. See `references/memory-vs-persistence.md`.

## Canonical repo layout OCC produces

```
<company_repo>/
  company_policies/            # PERSISTENCE — versioned, founder-facing
    charter.md                 # business VISION only (founder + CEO); no tech design
    org-chart.md               # the persona registry: role → skill → reviewer → reports-to
    job-descriptions/<role>.md # one per role; the recruiting spec
    roadmap.md                 # milestones + success criteria (CEO-owned)
    architecture.md            # current-system shape, seams, invariants (ARCHITECT-owned)
    data-model.md              # types, schema, persistence (ARCHITECT-owned)
    specs/<area>.md            # backend spec R#/A# (PM + ARCHITECT co-owned)
    system-design.md           # next-phase deployment, API, migration (TECH-LEAD-owned)
    use-cases/                 # step-by-step use-case catalog, backed by specs/ (PM-owned)
    ux/                        # UX manuals (UX-manager-owned)
    milestones/<id>.md         # per-milestone spec (CEO-owned)
    tasks/<id>.md              # per-task spec; acceptance = the spec's A# (tech-lead-owned)
    test-strategy.md           # acceptance/integration test plan (QA-owned)
    knowledge/                 # KNOWLEDGE tier — shared map + gotchas, HR-gardened
    decisions.md               # autonomous decision log + the outcome that harnessed each
    handbook.md                # handoff, review bar, consultation, escalation, automation
    performance/<date>.md      # HR reviews + failure-mode catalog + knowledge GC
  .claude/skills/              # the recruited org — invokable persona skills
    <company>-ceo/SKILL.md     +  <company>-board-reviewer/   # CEO's plans reviewed before founder
    <company>-product-manager/ +  <company>-pm-reviewer/
    <company>-ux-manager/      +  <company>-ux-reviewer/
    <company>-architect/       +  <company>-architect-reviewer/   # correctness (design)
    <company>-tech-lead/       +  <company>-tech-lead-reviewer/   # delivery (≠ architect)
    <company>-<executor>/      +  <company>-<executor>-reviewer/  # one reviewer per executor
    <company>-qa-engineer/SKILL.md                              # product QA (≠ the reviewers)
    <company>-hr/SKILL.md
    <persona>/MEMORY.md        # MEMORY tier, co-located with each persona
```

## How a built company runs (CEO command surface)

Once recruited, the founder talks to **one** persona — the CEO. The CEO exposes four commands (see the ceo job description / template):

- **review-and-plan** — an iterative, brainstorming-style session *with the founder* (superpowers:brainstorming discipline): review the current situation (progress, outcomes, performance), ask questions one at a time, and decide the next direction — changing course if the evidence says so. **This is where the founder's decision weight is paid up front:** every foreseeable direction-fork for the upcoming run is pre-decided or given a default + tolerance, so the run inherits standing authority and never stops to ask. The result is written back into the CEO-owned artifacts (roadmap, milestones, charter, decisions). This is the CEO's *product/direction* review (distinct from `occ-strategic-review`, which is about *org structure*).
- **report-architecture** — discuss architectural issues/options with the founder, adjust, log decisions.
- **report-progress** — read roadmap + ledger, report milestone status and what's next, plus the live decision trio: DIRECTION-tagged decisions, the open needs-list, and the current drift state.
- **run-the-company** — run the access pre-flight, then execute the next milestone end-to-end **long-running, never blocking on the founder**: decide everything outcome-harnessed; for a direction-changing fork, decide against the signed plan's default (cheapest-to-reverse), log it `DIRECTION`, keep running; for a human-only access gap, park only that thread and run on; emit a drift digest if the accumulated calls bend a charter non-negotiable or roadmap success criterion — and keep running. The founder course-corrects at the next `review-and-plan`, not mid-run.

The CEO sequences milestones inline. **Mid-level managers orchestrate execution via the Workflow tool** (fan out executors, run adversarial reviewer panels, converge). See `references/dynamic-workflows.md`.

## Core disciplines (non-negotiable, baked into every persona)

1. **Independent review at every layer** — no artifact (roadmap, use-cases, specs, architecture, data-model, task spec, code, tests) is done until an *independent* reviewer of that layer passes it — not the producer, not the orchestrator's own eyeball. The CEO's plans are reviewed by a board-reviewer before the founder; every producing role has a reviewer seat. Two levels of QA: the **reviewers** guard the agentic workflow against hallucination/spec-drift; the **QA engineer** verifies the real product (acceptance + integration vs the spec). Deadlines do not lower this bar. See `references/handoff-and-review.md`.
2. **Outcome-harnessed automation, long-running by construction** — make every reversible technical decision autonomously and bind it to a measurable outcome (the spec's acceptance); the *outcome* is the arbiter, not debate or sign-off. The founder's decision weight is paid **up front in planning** (foreseeable direction-forks pre-decided or given a default + tolerance), so the run **never hard-stops to ask**: a mid-run direction-fork is decided against the signed default and logged `DIRECTION`; an access gap parks only its thread; a drift digest notifies (it does not gate). Front-load every human-granted access before a long run. See `references/automation.md` (and `decision-escalation.md`).
3. **Spec-driven, design-before-build** — CEO owns roadmap+milestone success criteria; **PM owns step-by-step use-cases; PM+Architect co-own the backend spec (R#/A#); Architect owns architecture+data-model; Tech Lead owns task decomposition, deployment/system-design, and delivery.** Use-cases are backed by specs; tests are written against the spec's acceptance. No execution without an approved design and a written spec. See `references/specs-and-use-cases.md`.
4. **Three storage tiers** — authoritative facts/specs to `company_policies/`; shared map+gotchas to `knowledge/` (referenced, deduped, budgeted); short-term role feedback to persona `MEMORY.md`. Only HR promotes between tiers. See `references/memory-vs-persistence.md`.
5. **Consultation, not guessing** — when a spec/design is ambiguous or an upstream artifact looks wrong, a role consults the owner via a structured, ID-referenced artifact (agentic, not agile ceremony) instead of inventing an interpretation. See `references/consultation.md`.

## Workflow

```
Founder intent
   │
   ▼
occ-strategic-review ──► company_policies/{charter, org-chart, job-descriptions/}
   │
   ▼
occ-recruiting ──► .claude/skills/{<company>-*}  (personas + reviewers + hr)
   │
   ▼
invoke <company>-ceo  →  run-the-company  →  ships milestones
   │                                            │
   │                              reviewers + managers + founder feedback
   ▼                                            ▼
occ-strategic-review (structural)        <company>-hr (incremental: memory → skill)
```

**REQUIRED SUB-SKILLS:** `occ-strategic-review`, `occ-recruiting`.
**REQUIRED BACKGROUND:** superpowers:writing-skills (recruiting writes persona skills), superpowers:brainstorming (strategic review elicitation).
**REFERENCES:** `references/` covers the corporate model, handoff/review contract (+ two-level QA), specs-and-use-cases, storage tiers (persistence/knowledge/memory), automation (outcome-harnessed), consultation, decision-escalation, dynamic-workflow orchestration, and artifact layout. **TEMPLATES:** see `templates/`. **WORKED EXAMPLE:** `examples/ledger-org/`.

---
> Source: [kezhu2008/occ](https://github.com/kezhu2008/occ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

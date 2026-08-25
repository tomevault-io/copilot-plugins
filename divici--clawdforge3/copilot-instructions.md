## clawdforge3

> This repository is **clawdForge 3.0**.

# CLAUDE.md

## Project Identity

This repository is **clawdForge 3.0**.

clawdForge 3.0 is an AI-native **software factory** that turns a user’s prompt into a working app or web app end to end.

It is **not**:
- a generic chatbot
- a generic agent playground
- a flashy AI demo
- a loose set of prompts

It **is**:
- a practical builder tool
- a trustworthy software factory
- a structured execution system
- a product that produces code **and** durable artifacts

Always optimize for:
- practical usefulness
- trustworthy state/progress
- strong product structure
- consistent output quality
- builder-focused UX

---

## Source of Truth

When making decisions, use the following priority order:

1. `PRD.md`
2. `Findings.md`
3. `designs/` assets and notes
4. existing code and architecture patterns already established in the repo

Do not invent a direction that conflicts with the PRD or Findings.

If something is ambiguous:
- prefer the more builder-focused interpretation
- prefer real state over theatrical UX
- prefer structured workflows over free-form prompting
- prefer durable artifacts over temporary chat output

---

## Core Product Model

clawdForge 3.0 has two primary modes:

### Interactive Mode
The system collaborates with the user before and during early planning:
- asks clarifying questions
- shapes the product direction
- creates a spec
- proposes architecture/stack
- creates an implementation plan
- gets approval at key checkpoints
- then builds mostly autonomously

### Autonomous Mode
The system:
- interprets the prompt
- uses the default stack and default policies
- minimizes interruption
- proceeds mostly autonomously
- only interrupts on true blockers, critical missing input, or high-risk decisions

Both modes should converge into the same core factory pipeline.

---

## Architectural Intent

The intended architecture is:

- **Web app** = control surface
- **Local runner** = real execution on the machine
- **LangGraph** = workflow orchestration, durable state, checkpoints, interrupts/resume
- **Claude Agent SDK** = execution engine inside workflow nodes

### Rule of Responsibility
- LangGraph decides **what happens next**
- Claude Agent SDK does **the task-level implementation work**
- the runner touches **files, shell, installs, builds, tests, git, previews**
- the UI shows and controls **the factory**

Do not collapse these responsibilities together unless there is a very strong reason.

---

## Workflow Philosophy

The system should follow a staged workflow, not “prompt in, random build out.”

Canonical flow:

1. intake
2. clarification / discovery
3. spec generation
4. architecture / stack decision
5. implementation plan
6. scaffold / foundation
7. implementation loop
8. verification loop
9. packaging / handoff

Always prefer:
- explicit stages
- visible progress
- durable artifacts
- clean handoffs between stages

Avoid giant uncontrolled agent loops.

---

## Worker Philosophy

Use a **small set of meaningful workers**.

Preferred worker types:
- Strategist
- Architect
- Planner
- Implementer
- Reviewer
- Fixer
- Packager

Workers exist for:
- context isolation
- responsibility boundaries
- tool boundaries
- execution clarity

Do **not** create an org-chart simulation with unnecessary worker roles.

Avoid agent theater.

---

## Skill Philosophy

Skills should be:
- modular
- task-specific
- dynamically loaded
- mostly invisible to the user

Good examples:
- clarify-product
- choose-stack
- write-spec
- create-plan
- scaffold-default-stack
- implement-auth
- build-ui
- run-verification
- fix-build
- package-handoff

Do not rely on one giant universal prompt when a smaller scoped skill is better.

Do not load unnecessary skills by default.

---

## UX Principles

The UX should feel:
- practical
- builder-focused
- trustworthy
- alive
- structured
- premium without being flashy

### Home Experience
The app should open into a **clean project dashboard**, not a dramatic cockpit.

### Forge Workspace
The forge workspace is the execution environment entered once a run starts.

It should:
- emphasize phase/status first
- feel immersive and builder-focused
- preserve clawdForge’s dark industrial amber-accented identity
- expose deeper developer detail in organized views

### Trust Rule
The UI must always communicate real state, not vague “AI thinking” language.

The user should be able to quickly understand:
- what is being built
- current phase
- active task
- what completed
- what is blocked
- whether intervention is needed
- whether the app currently runs

### Live Preview
A Live App / preview surface is important.
Treat preview as proof that the factory is producing something real.

---

## Design Guardrails

Use the existing designs and notes as reference.

The desired visual direction is:
- dark near-black base
- restrained amber/orange accent
- industrial / premium / practical tone
- closer to clawdForge 2.0 than generic SaaS dashboards

Do not drift toward:
- bright blue SaaS styling
- playful or gamified UI
- flat generic productivity tooling
- noisy AI-dashboard aesthetics

### Important Forge Workspace Note
The Forge Workspace is sensitive to over-refinement.
Making it “cleaner” too aggressively can remove clawdForge’s identity.

When refining that screen:
- improve hierarchy
- improve state clarity
- improve preview relevance
- preserve atmosphere
- preserve dark immersive framing
- preserve builder-focused tone

---

## Artifact Discipline

The product must produce durable artifacts, not just code.

Each run should persist and expose artifacts such as:
- original prompt
- clarified requirements
- selected mode
- architecture decisions
- generated spec
- generated implementation plan
- task list and statuses
- logs / activity
- changed files / diffs
- verification results
- issues / blockers
- final handoff summary

When implementing features, think in terms of **artifact lifecycle**, not just UI screens.

---

## Default Stack Direction

For autonomous mode, prefer a strong default path.

Current default web stack:
- Next.js
- TypeScript
- Tailwind
- shadcn/ui
- Prisma
- Postgres

Interactive mode may override the stack when justified.

Do not prematurely expand into many stacks before the core system is coherent.

---

## Engineering Preferences

Prefer:
- TypeScript end-to-end where reasonable
- clean boundaries between UI, orchestration, runner, and artifacts
- strongly typed state and data models
- explicit status enums over vague booleans
- reusable internal modules over giant files
- deterministic structures where possible
- practical logs and telemetry

Avoid:
- tightly coupling UI to raw execution internals
- hiding important state transitions
- magical behavior with no visible artifact trail
- giant monolithic prompts with no stage separation

---

## Build Behavior Expectations

For complex changes:
- reason from the PRD and Findings first
- make or update a plan before large implementation changes
- keep work aligned with the staged factory model
- preserve consistency with the overall product direction

When making UX decisions:
- optimize for clarity and trust first
- then depth
- then style

When making architecture decisions:
- optimize for durable state, structured execution, and inspectability

---

## Non-Goals

Do not turn clawdForge 3.0 into:
- a general-purpose chat app
- a random multi-agent lab
- an “AI teammate simulator”
- a purely visual concept piece
- a terminal wrapper with weak product structure

---

## Contribution Rule

Every meaningful implementation decision should strengthen at least one of these:
- trust
- structure
- visibility
- consistency
- builder usefulness
- artifact quality

If a change makes the app more impressive but less trustworthy or less usable, reject it.

---

## When Unsure

When uncertain, prefer:
- simpler workflows
- clearer state
- better artifacts
- fewer workers
- modular skills
- stronger defaults
- practical UX
- real proof of progress

---
> Source: [Divici/clawdforge3](https://github.com/Divici/clawdforge3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->

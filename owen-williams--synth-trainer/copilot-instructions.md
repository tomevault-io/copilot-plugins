## synth-trainer

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

Mobile Synthesis Trainer — a mobile-first, PWA-installable ear-training app for subtractive
synthesis (React + TypeScript frontend, Express + TypeScript API, PostgreSQL). This is a proof
of concept validating that phone-based, target-matching sound design lessons can teach real
auditory intuition in short (1-3 minute) sessions.

There are two learning experiences in the app, and they deliberately share nothing but the
shell, the storage layer and the design system:

- **Foundations** — the original Syntorial-style course. Hear a target, move the controls to
  match it, submit for a weighted parameter score. Specified in [`docs/spec.md`](docs/spec.md).
- **Sound Lab** — guided builds teaching experimental club synthesis (pings, metal, croaks,
  growls, tears). No target patch and no score anywhere: each step names a *percept* to find
  ("turn it up until the wobble becomes a croak") and unlocks when the learner is in that
  region. Specified in [`docs/lab-spec.md`](docs/lab-spec.md). It has its own patch model
  (`LabPatch`, a fixed signal chain), its own always-running AudioWorklet voice, and its own
  content model. **Do not generalise Foundations machinery to serve it, or vice versa** —
  §39 of `docs/spec.md` records why.

Unresolved product/design questions live in [`docs/open-questions.md`](docs/open-questions.md) —
check it before re-opening a debate that is already recorded there, and update it when one is
settled.

**The full product/technical spec lives at [`docs/spec.md`](docs/spec.md). Read it before
making any non-trivial change** — it defines the curriculum, the synth patch schema, the
exercise-definition model, the scoring model, the API surface, the database schema, and phased
build order (vertical slice → persistence → curriculum engine → PWA/offline → polish). The spec
is a living document: when a real decision supersedes something written there (e.g. an ORM or
library choice), update the spec in place rather than letting it drift out of sync with the
code. Section 39 of the spec ("Implementation decisions log") records technology choices the
original spec deliberately left open — check it before introducing a new library that
overlaps with an existing choice.

Core design principle (spec §38): the product is the loop **listen → adjust → compare → submit
→ understand → retry**. When a decision trades off synthesis flexibility against teaching
clarity, teaching clarity wins. Fewer visible controls, not more.

## Model-specific operating rule: Fable orchestrates, Opus implements

When the active model for this session is **Fable**, Fable must act purely as an orchestrator
and must NOT write or edit application code, tests, or infrastructure files directly.

- Fable may: read files, plan, break work into tasks, write/update documentation (this file,
  `docs/spec.md`, README files), and orchestrate implementation via subagents — using the
  `Agent` tool or the `Workflow` tool with agent `model: 'opus'`.
- Fable must NOT: use `Write` or `Edit` to author source code, config, migrations, or tests
  itself. Any actual programming task — new features, bug fixes, refactors, schema changes,
  tests — must be delegated to an Opus-model agent.
- This applies regardless of task size. Even a "quick fix" gets delegated.
- If Fable is genuinely unsure whether something counts as "programming" (e.g. editing a
  `package.json` dependency list), default to delegating it.

This rule is specific to Fable. Other models (e.g. Sonnet, Opus) invoked directly as the main
session model should follow normal Claude Code behavior and are not bound by this restriction.

## Architecture reminders (see spec for full detail)

- Monorepo: `/apps/web` (React/Vite PWA), `/apps/api` (Express), `/packages/*` (shared domain
  types, scoring logic, synth patch schema, validation), `/database` (migrations + seeds).
- The synth engine runs **entirely client-side** (Web Audio API). The server never generates or
  streams audio — it only serves curriculum data and validates/persists scores.
- Scoring logic must be shared between client (provisional/instant feedback) and server
  (authoritative validation) — do not fork the implementation.
- The server never trusts client-submitted scores, pass/fail status, or target values; it always
  recalculates from the stored exercise definition (spec §23).
- Learners are anonymous by default (opaque client-generated ID) — no registration required for
  the full POC flow (spec §20).

---
> Source: [owen-williams/synth_trainer](https://github.com/owen-williams/synth_trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->

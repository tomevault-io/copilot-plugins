## cadence-flow

> Guidance for every Claude session in this project. Read this first; the details live

# CLAUDE.md — Cadence Flow

Guidance for every Claude session in this project. Read this first; the details live
in the linked docs.

## What this project is

A project run on the **Cadence Flow** lifecycle: numbered skills in `.claude/skills/`
carry work from idea → plan → review → design → build → verify → ship → reflect.
The full workflow narrative is [docs/workflow.md](docs/workflow.md).

The person you're working with is building software for the first time. Explain
decisions in plain language, define jargon on first use, and when they seem stuck on
*what* to build rather than *how*, suggest `/1-office-hours` rather than writing code.

## The laws

1. **Proposals first.** No feature gets built without a proposal in
   `docs/proposals/`. Small fixes and chores are exempt; anything with scope is not.
   Author proposals from [docs/proposals/TEMPLATE.md](docs/proposals/TEMPLATE.md).
2. **The folder is the status.** Proposal state = its lifecycle folder
   (`1-backlog → 2-drafts → 3-implementing → 4-implemented → 5-closed`). Change state
   with `git mv`, never by editing a status field or leaving a copy behind.
3. **Rigor is proportional to reversibility.** Default posture is good-enough: ship
   and iterate. **One-way doors** — persisted data shapes, API contracts, public
   names — get the careful review before they land (see the class system in
   [docs/proposals/README.md](docs/proposals/README.md)).
4. **Verify before declaring done.** A proposal's definition-of-done says how it's
   verified; run that verification before moving it to `4-implemented`. "It should
   work" is not verification.
5. **Record honestly.** When reality deviates from the plan, the proposal's Outcome
   section says so. A tidy but false record is worse than none.

## Where things go

- Proposals: `docs/proposals/<lifecycle-folder>/<slug>.md`
- Clarity docs, design notes, QA/audit reports: `docs/working-docs/`
- Retro snapshots + learnings: `docs/retros/`
- Design direction (once one exists): `DESIGN.md` at the project root

## Git

Commit messages: imperative subject ≤ 72 chars; body explains *why* when non-obvious.
Never commit, push, or merge without being asked — except where a skill's flow
explicitly includes it (invoking `/7-ship` is that ask, for its scope).

---
> Source: [mikesmayer/cadence-flow](https://github.com/mikesmayer/cadence-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->

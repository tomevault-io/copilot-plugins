## harness-build-skill

> Start here. This repo builds reusable Harness systems.

# AGENTS.md

Start here. This repo builds reusable Harness systems.

## Read First

- `ARCHITECTURE.md` - repo layers and why they exist
- `docs/design-docs/core-beliefs.md` - the main design beliefs for this project
- `docs/generated/harness-profile-template.md` - the profile schema that drives the generic bootstrap
- `docs/product-specs/harness-from-profile.md` - the main reusable deliverable in this repo
- `docs/exec-plans/index.md` - active/completed plan system for this repo

## Default Working Loop

1. Read the task.
2. Read `ARCHITECTURE.md` and `docs/design-docs/core-beliefs.md` before changing the skill or template structure.
3. If the task is non-trivial, create or update a plan in `docs/exec-plans/active/`.
4. Keep generic Harness logic separate from project-specific examples.
5. If you change the profile schema, update the skill references and template assets together.
6. If you change generated output structure, update both the skill docs and the template files.
7. Before finishing, update the active/completed plan record.

## Source Of Truth

- design intent: `docs/design-docs/`
- generated input template: `docs/generated/`
- product deliverable docs: `docs/product-specs/`
- references and source material: `docs/references/`
- execution state: `docs/exec-plans/`
- reusable skill implementation: `harness-from-profile/`

## Working Rules

- Treat the Harness operating model as reusable infrastructure.
- Treat MinT-like examples as examples, not hard-coded product truth.
- Keep `AGENTS.md` short and keep details in the mapped docs.
- Prefer stable file roles over ad hoc note files.
- Record design changes in repo docs, not only in chat.

---
> Source: [lysrain21/Harness-build-skill](https://github.com/lysrain21/Harness-build-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

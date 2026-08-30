## agents-md-harness

> This `AGENTS.md` is for working on the `agents-md-harness` repository itself.

# AGENTS.md

This `AGENTS.md` is for working on the `agents-md-harness` repository itself.
It is **not** the template that end users generate with `npx agents-md-harness setup`.

## Task classification

Before acting, classify the task:

- **CLI development** → modifying bin/, testing, packaging
- **Template work** → editing files under `template/`
- **Documentation** → README, repo docs
- **Release** → versioning, changesets, publishing

Then load only the minimum relevant documents from `_harness/`.

## Routing

- Project overview → `_harness/readme.md`
- Structure lookup → `_harness/catalog.md`
- Maintainer constraints → `_harness/rules.md`
- Task routing → `_harness/routing.md`
- Development workflows → `_harness/workflow.md`

## Loading principles

- Root `AGENTS.md` + `_harness/` = maintainer instructions
- `template/` = user-facing output
- Do not mix the two
- Load smallest useful instruction set

---
> Source: [chioio/agents-md-harness](https://github.com/chioio/agents-md-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->

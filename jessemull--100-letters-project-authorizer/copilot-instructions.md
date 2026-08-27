## 000-context

> Mandatory governance context loading and non-negotiable constraints before any repository changes.


# Repository Context Loading

Context loading is **not optional** — it is the first step before any work.

## Before ANY Changes

Read in order:

1. `CONTEXT.md`
2. `AGENTS.md`
3. Mandatory docs from CONTEXT.md: `docs/GOVERNANCE.md`, `docs/ARCHITECTURE.md`, `docs/TESTING.md`, `docs/COMMENTS.md`, `docs/SECURITY.md`, `docs/DEPENDENCIES.md`, `docs/RELEASES.md`, `docs/CI_CD.md`

## MUST

- [ ] Read `CONTEXT.md` first
- [ ] Read `AGENTS.md` second
- [ ] Read mandatory docs (do not skip as "unrelated")

## MUST NOT

- Skip context loading
- Selectively read only "relevant" docs
- Begin changes before loading is complete

## Precedence

CONTEXT.md > GOVERNANCE.md > ARCHITECTURE.md > domain docs > inline comments

## Non-negotiables (summary)

- Lambda TOKEN authorizer; Node 24; fail closed
- TypeScript strict; ≥80% Jest coverage
- Conventional Commits + Husky
- No hardcoded secrets
- Quality gate: `make preflight`

---
> Source: [jessemull/100-letters-project-authorizer](https://github.com/jessemull/100-letters-project-authorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->

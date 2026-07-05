## cv-builder

> Open source, privacy-first resume evaluator. This repo doubles as a **power-user

# CV Builder

Open source, privacy-first resume evaluator. This repo doubles as a **power-user
pack**: clone it, open Claude Code here, and evaluate your resume locally with
your own agent — nothing leaves your machine.

## Power-user commands

- `/evaluate-cv <resume.pdf|.md|.txt> [--jd <job.md>]` — score a resume against
  the rubric and surface issues + unsupported claims.
- `/setup-profile` — assemble a resume if you don't have a file ready.

## No build needed for evaluation

The `cv-evaluation` skill (`.claude/skills/cv-evaluation/`) reads the prompts in
`packages/prompts/prompts/` and the rubric/archetypes in
`packages/intelligence/src/` straight from the repo, and returns an `EvalResult`
validated against `packages/schemas`. `pnpm install` is only needed to run the
package tests or the web app — not for the power-user flow.

## Repo shape

pnpm + turbo monorepo. `packages/schemas` (contract) → `packages/intelligence`
(rubric + archetypes) → `packages/prompts` (the prompt pack) → the skill ties
them together.

---
> Source: [TechImmigrants/cv-builder](https://github.com/TechImmigrants/cv-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->

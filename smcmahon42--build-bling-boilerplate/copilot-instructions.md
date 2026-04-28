## build-bling-boilerplate

> > **For the setup agent:** if you are reading this right after cloning the template, stop here and run `/bootstrap-project` from `.claude/skills/bootstrap-project.md`. It will replace the placeholders, trim sections, and seed your project.

# CLAUDE.md — {{PROJECT_NAME}}

> **For the setup agent:** if you are reading this right after cloning the template, stop here and run `/bootstrap-project` from `.claude/skills/bootstrap-project.md`. It will replace the placeholders, trim sections, and seed your project.

This file is a **router**, not a manual. Claude should read it on every session, then fetch the topic file(s) relevant to the current task. Keep it slim — detailed guidance lives in `claude-instructions/`.

---

## Project

- **Name:** {{PROJECT_NAME}}
- **One-liner:** {{PROJECT_DESCRIPTION}}
- **Primary stack:** {{PRIMARY_STACK}}
- **Status:** {{PROJECT_STATUS}}  <!-- e.g. exploring, alpha, production -->

## House rules (non-negotiable)

1. **TDD is the starting posture.** Every change begins with a failing test. Code follows. See `claude-instructions/testing-practices.md`.
2. **Security is a first-class gate.** OWASP Web Top 10, OWASP LLM Top 10, and OWASP Agentic Top 10 all apply. See `claude-instructions/security-practices.md` and run `/security-review` before shipping sensitive changes.
3. **Every change ships documentation.** If the change affects behavior, architecture, or operations, docs go in the matching bucket. See `claude-instructions/documentation-discipline.md`.
4. **Architectural decisions get an ADR.** See `docs/decisions/README.md` and run `/new-adr` to scaffold one.
5. **No business-specific code in shared layers.** Domain logic stays in its owned module.
6. **Trust the tests and the types; don't add validation that can't fail.** Validate at system boundaries only.

## Process checkpoints (enforce these, don't just agree with them)

Each task follows the same loop. These aren't aspirational — they're the default shape of work in this repo.

1. **Orient.** Read `CLAUDE.md` (you are here), then the matching topic file(s) from the table below. Scan `.claude/skills/README.md` — the skills index — to pick relevant skills *before* you start editing.
2. **Scope.** Classify the task as small / medium / large (see [`claude-instructions/development-workflow.md`](claude-instructions/development-workflow.md#task-scale-matrix)). The scale determines which docs and steps are required.
3. **Propose.** For medium or large tasks, share the plan (what files, what shape, what tests) *before* editing. For small tasks, proceed directly.
4. **Test-first.** Write the failing test. Run it. Confirm the failure is the one you expect.
5. **Edit.** Smallest possible change to green the test. No unrelated cleanup.
6. **Verify.** Run the full local quality gate: tests, linters, pre-commit hooks, and the relevant OWASP-aware review if security-sensitive.
7. **Document.** Update the matching doc bucket in the same change (ADR, README, runbook).
8. **Commit.** Conventional Commits. One logical unit per commit. See [`claude-instructions/commit-conventions.md`](claude-instructions/commit-conventions.md).

For destructive actions (deletions, force-pushes, production writes, external messages, schema drops), stop and confirm before acting — even inside a larger approved plan.

## When you're about to work on…

| Task | Read |
| --- | --- |
| A bug fix, feature, or refactor | `claude-instructions/development-workflow.md` |
| A commit or PR | `claude-instructions/commit-conventions.md` |
| Tests (unit, integration, e2e, Playwright) | `claude-instructions/testing-practices.md` |
| Security review or threat modeling | `claude-instructions/security-practices.md` |
| Adding a dependency | `claude-instructions/dependency-vetting.md` |
| Creating a module or boundary | `claude-instructions/component-explainability.md` |
| An architectural decision | `docs/decisions/README.md` |
| Planning an epic or filing issues | `claude-instructions/epics-and-projects.md` |
| Writing or editing Claude memory | `claude-instructions/claude-memory.md` |
| Splitting CLAUDE.md as the project grows | `claude-instructions/scaling-claude-instructions.md` |
| Writing or organizing docs | `claude-instructions/documentation-discipline.md` |

## Growing the instruction system

This file is **Stage 2** in a three-stage pattern — single `CLAUDE.md` → slim router + `claude-instructions/` → per-module routers. Signals that it's time to split: topic files over ~300 lines, modules with distinct stacks, onboarding that takes more than one read of this file. See `claude-instructions/scaling-claude-instructions.md`.

## Project-specific conventions

<!-- Fill in during /bootstrap-project. Delete this section if there is nothing project-specific yet. -->

- ...

---
> Source: [smcmahon42/build_bling_boilerplate](https://github.com/smcmahon42/build_bling_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->

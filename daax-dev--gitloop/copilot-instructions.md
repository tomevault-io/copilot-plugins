## gitloop

> GitHub Copilot reads this file automatically. Rules here are enforced in every session.

# Copilot Instructions

GitHub Copilot reads this file automatically. Rules here are enforced in every session.

---

## Project
Name: gitloop
Purpose: Local, git-only inner-loop tool — AI coding models/tools hand work off event-driven and deterministically via native git primitives (tags, worktrees). Pure TypeScript. No CI/CD. See OBJECTIVE.md.

---

## Operator Preferences
<!-- Operator-specific. Revise or replace when applying to a different operator. -->
- State facts only. No sugarcoating.
- Surface problems, blockers, and risks immediately.
- Consult before one-way-door or architectural decisions.
- Never answer from a guess. Say so when a claim cannot be validated.
- Objective language. No first-person pronouns. No apologies.

---

## Planning
- A plan is required for any non-trivial change. Trivial = typo fix, single-line config update, obvious rename.
- Write the plan first. Present it. Wait for approval. Do not start coding until approved.
- Present options with trade-offs. The operator decides; the agent executes.

---

## Stack
- Runtime: Node 22 LTS (pinned via `.nvmrc`). TypeScript only.
- Package manager: pnpm — no npm or yarn.
- Test framework: Vitest. Run TS directly with tsx.
- CI: none (local inner-loop only, by design).

---

## Code Conventions
- Run the formatter before committing. No hand-formatted code.
- All tests must pass before declaring done.
- No `any` / untyped code without a justifying comment.
- Lockfiles are committed. Updating a lockfile is a deliberate change — note it in the PR.
- Generated code lives under an excluded path. Never edit generated files by hand.
- Pre-commit hooks gate formatter and linter. No bypasses (`--no-verify`).

---

## Source Control
- Never commit directly to `main`. All work lands via PR.
- Branch naming: `feature/`, `fix/`, `docs/`, `chore/`.
- Commits: imperative mood, present tense. Subject ≤ 72 characters. Body explains **why**.
- PR body must include: problem statement, approach, alternatives considered, test evidence.
- Never merge your own PR unless explicitly authorized.
- Never commit secrets, tokens, keys, or `.env` files with live values.

---

## Architecture
- Module boundary = test boundary. If two modules cannot be tested apart, they are one module.
- Secrets go through the platform secret store. Never in source control or env files.
- UTC everywhere internally. Local time is a presentation concern.
- "Temporary" workarounds without an expiry date and an owner are not acceptable.
- No shared databases between services. Share data via APIs or events only.

---

## Definition of Done
A task is done only when:
- All tests pass.
- Formatter and linter pass with no errors.
- PR opened with problem statement, approach, and test evidence.
- No `[FILL IN]` placeholders left in affected files.
- Decisions logged in `.logs/decisions/` if a non-trivial choice was made.

---
> Source: [daax-dev/gitloop](https://github.com/daax-dev/gitloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->

## codex-starter

> Working agreements for AI coding agents (Codex CLI, etc.).

# AGENTS.md

Working agreements for AI coding agents (Codex CLI, etc.).
Keep this file short. Read referenced docs only when they are relevant to the current task.
Keep repo-specific rules under **Project-specific configuration**.

## Prime Directive

Ship **production-ready** changes with:

- minimal risk
- clear intent
- verifiable behavior
- no hidden breakage

If you cannot verify something, say so and provide exact steps the user can run.

---

## Default Autonomy And Safety

- Start with read-only exploration before editing.
- Keep changes inside the repo.
- Do not modify dotfiles, global configs, or user machine state unless explicitly instructed.
- Avoid bypassing approvals or safeguards unless explicitly requested.
- Prefer the smallest safe, durable change at the right abstraction level.
- When requirements are unclear, ask targeted questions or proceed with the safest labeled assumption.

---

## Technical Judgment

- Do not agree with weak ideas by default.
- Push back when a proposal is brittle, overcomplicated, hard to verify, or likely to create maintenance debt.
- Push back only with a concrete technical reason, not taste.
- Distinguish evidence from inference.
- When disagreeing, name the risk and propose a better alternative.

---

## Discussion And Execution Protocol

The user prefers a numbered discussion protocol before execution when something still needs to be resolved.
Use it to present blocking decisions, risks, or questions.
Keep the list short, merge related concerns, and use short prose for non-blocking updates.

Do not begin execution until blocking numbered items are resolved.
If the request is broad or underspecified, stop after a short clarification or planning checkpoint instead of jumping into code.

For details, read `.agent/docs/workflow.md`.

---

## ExecPlans

Use an **ExecPlan** for complex features or significant refactors.

- `.agent/PLANS.md` is the source of truth for ExecPlan format and process.
- Do not invent a separate planning framework.
- The ExecPlan index lives at `.agent/execplans/INDEX.md`.
- If an ExecPlan type is unspecified, ask: `What type is this ExecPlan: draft, active, or archive?`

---

## Continuity

If this repo uses `.agent/CONTINUITY.md`, read it at session start, after compaction, when resuming paused work, or when the user references prior state.
Do not reread it every turn.
Update it only when there is a meaningful delta in goal, constraints, decisions, state, open questions, working set, or important tool outcomes.

For format and maintenance rules, read `.agent/docs/continuity.md`.

---

## Implementation Rules

- Make the smallest durable change that solves the problem at the right abstraction level.
- Preserve sound existing style and architecture, but do not preserve weak or accidental structure just because it exists.
- Prefer focused diffs over broad rewrites, but make modest structural improvements when they materially reduce future churn.
- Write self-explanatory code; avoid comments unless they are truly necessary.
- Fail fast; do not swallow errors silently.
- Do not reinvent standardized functionality without checking whether a mature dependency is the better choice.

For detailed code, editing, and dependency guidance, read `.agent/docs/implementation.md`.

---

## Host Package Policy

- Never install system packages on the host unless explicitly instructed.
- Prefer project-local tooling and repo-local scripts.
- If tooling is missing, propose options and ask before touching host system state.

---

## Secrets And Sensitive Data

- Never print secrets to terminal output.
- Do not ask users to paste secrets.
- Avoid commands that may expose sensitive data unnecessarily.
- Redact sensitive strings in displayed output.

---

## Reference Docs

Read these on demand, not by default:

- `.agent/PLANS.md` for ExecPlan authoring and lifecycle
- `.agent/docs/workflow.md` for workflow, research, and verification guidance
- `.agent/docs/implementation.md` for code standards, editing rules, and dependency choices
- `.agent/docs/continuity.md` for continuity ledger rules

---

## Project-Specific Configuration

Fill these in for each repo:

- Build:
- Lint:
- Test:
- Typecheck:
- Docs:
- Run/Dev:
- Deployment or migration constraints:
- Other repo-specific invariants:

---
> Source: [Skarian/codex-starter](https://github.com/Skarian/codex-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->

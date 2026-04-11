## coding-agent

> This file contains project-level rules for `/Users/wrr/work/coding-agent`.

# Repository AGENTS Rules

This file contains project-level rules for `/Users/wrr/work/coding-agent`.
These rules are lower priority than system/developer/runtime policies and higher priority than ad-hoc preferences in normal prompts.

## Scope

- Applies to this repository and all subdirectories.
- Prefer minimal, local changes over broad refactors.
- Do not touch unrelated modified files unless the task explicitly requires it.

## Prompt Layering in This Repo

- System directives: truthfulness, safety, web-verification boundaries, runtime policy compliance.
- Developer directives: tool usage discipline, editing workflow, review/verification output format.
- AGENTS.md (this file): repository conventions, commands, workflow expectations, branch/worktree policy.

## Repository Conventions

- Primary implementation area: `src/agent-v2/**`.
- Prompt construction code lives in `src/agent-v2/prompts/**`.
- Keep prompt text explicit and deterministic; avoid ambiguous wording.
- When changing prompt rules, preserve backward-compatible behavior unless the task explicitly asks for behavior change.

## Verification Commands

Run focused checks for prompt changes:

```bash
pnpm test:run -- src/agent-v2/prompts/__tests__/operator.test.ts
```

For broader validation when changes are non-trivial:

```bash
pnpm typecheck
pnpm lint
pnpm test:run
```

If a check is skipped due to time/tooling constraints, state it explicitly.

## Git and Branch Policy

- Branch naming convention: `codex/<topic>`.
- Never use destructive git operations unless user explicitly asks.
- Never discard unrelated local changes.

## Worktree Policy (Project Specific)

Use a dedicated git worktree when any of the following is true:

- parallel feature/fix work is needed;
- change is high-risk and isolation is useful (large refactor, migration, wide prompt rewrite);
- different branches must be tested in parallel.

Do not create a worktree for small, single-branch edits.

Suggested workflow:

```bash
git worktree add -b codex/<topic> ../coding-agent-<topic>
```

Remove after merge/cleanup:

```bash
git worktree remove ../coding-agent-<topic>
git worktree prune
```

## Code Review Expectations

- Findings first, ordered by severity.
- Include file paths and line numbers when possible.
- Mention testing gaps and residual risks explicitly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/renxqoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/renxqoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

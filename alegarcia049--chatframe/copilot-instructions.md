## chatframe

> - Do not invent files, scripts, APIs, or behaviors.

# AGENTS.md — Repo instructions for coding agents (Codex)

## Prime Directive

- Do not invent files, scripts, APIs, or behaviors.
- Do not ask for confirmation, proceed with further ado.
- Prefer small, safe changes with clear verification.
- If critical info is missing, ask at most ONE question; otherwise proceed conservatively and state assumptions.
- If a new function is introduced to replace an existing one, the old one must be removed in the same change; the PR fails if duplicates remain.
- Do not mention PR/commit/CI unless the repository contains scripts/configs for it or it is explicitly requested.
- Do not describe workflows (e.g., webhook/queue/worker) unless they are evidenced by concrete files in the repo. If it’s not clear, write ‘Not determined’.

## Read-first workflow (mandatory)

When asked to change or document anything:

1) Scan the repository to understand structure, entrypoints, scripts, and conventions.
2) Propose a short plan and an outline (index) BEFORE making changes.
3) Then implement changes and provide results in the requested format.

## Output format

- If modifying code: output a **unified diff** (git patch) unless the user requests otherwise.
- If generating docs: output full file contents with a header `FILE: <path>` for each file.

## Scope control

- Only modify files explicitly allowed by the user.
- If scope is not provided, modify the minimum set of files needed and list them in the plan.

## Upgrade / Migration rules (anti “add without removing”)

When the task is a version upgrade, API migration, or refactor:

- **Replace, don’t wrap:** do not add a new API while keeping the old one unless explicitly requested.
- **No dead code:** remove obsolete functions/exports and clean imports/exports.
- **No duplicate surface:** do not leave `foo` and `fooV2` side-by-side unless a temporary compatibility layer is explicitly required.
- Compatibility layers are allowed ONLY if:
  1) required by external callers,
  2) clearly marked `@deprecated`,
  3) include a removal plan (issue/date),
  4) have tests for both paths.

## Acceptance checklist (always apply)

- Build/typecheck must pass using repo scripts if available (e.g., `npm run build`).
- Tests must pass if configured.
- After migration: search/grep for removed APIs/strings should be 0 results (or exceptions justified).
- No leftover TODOs or commented-out code related to the task.

## Documentation tasks

When asked to document the repo, generate:

- `STRUCTURE.md` (stack, tree, responsibilities, data flow, conventions)
- `TOOLS.md` (setup, scripts, env vars names only, troubleshooting, upgrade mode rules)
  If tests/lint are missing, document as “not configured”.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alegarcia049)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Alegarcia049)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

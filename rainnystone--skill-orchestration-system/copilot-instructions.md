## skill-orchestration-system

> generates `sos-*` pointer skills, and creates backups before writes.

# Agent Instructions

## Project Overview

Skill Orchestration System (SOS) is a Python CLI for organizing agent skills into
auditable, activatable packs. It scans skill folders, proposes functional packs,
writes reviewable plans, applies migrations only with explicit confirmation,
generates `sos-*` pointer skills, and creates backups before writes.

This is the canonical public repository. Public files must stay free of personal
paths, private workspace details, tokens, account data, and local planning logs.
Local NHK, planning, spec, and archive surfaces may exist in this checkout, but
they are intentionally ignored by Git.

## Workspace Navigation

Start with the smallest current surface that can route the task safely:

1. `AGENTS.md` for stable execution rules.
2. `coding-agent-guide.md` for local task routing, when present.
3. `documentation-governance.md` for local active/archive rules, when present.
4. `task_plan.md`, `findings.md`, and `progress.md` only when they are active.
5. `README.md`, `README_CN.md`, `references/`, `src/`, and `tests/` for public product work.

Ignored local workflow files include `task_plan.md`, `progress.md`,
`findings.md`, `coding-agent-guide.md`, `documentation-governance.md`, `docs/`,
and `archive/`. They are useful for local continuity but must not be committed.

## Architecture Boundaries

- CLI writes must stay deterministic, auditable, and dry-run-first.
- `scan`, `propose`, and dry-run commands must not mutate active skill roots.
- Config, manifest, vault, backup, restore, sync, and source deletion behavior must be covered by tests.
- Source deletion must remain opt-in and require explicit confirmation.
- Pointer skills should remain short routing surfaces; full skill content belongs in the managed vault.
- Do not hardcode local absolute paths or user-specific defaults.
- Host-specific disable behavior must stay isolated to operation kinds (`MOVE_TO_ARCHIVE` for Claude, `DISABLE_CODEX_SKILL` for Codex) and validated in `apply._validate_plan` before any write.

## Mandatory Execution Rules

### Immutable Data Patterns
- Never mutate state objects. Always return new copies.
- `const newState = { ...oldState, field: newValue }`
- Array operations: `slice`, `map`, `filter` — never `splice`, `push`, `sort` in-place.

### Deterministic Boundaries Must Stay Deterministic

- Validation, schema checks, path checks, reference integrity, and persistence guarantees should be handled by deterministic code.
- Do not delegate deterministic validation to heuristic or probabilistic flows unless the human explicitly approves that tradeoff.

### Keep Rules, Code, Tests, and Active Docs in Sync

- If code, tests, and active project documents disagree, resolve intended behavior first.
- Then bring implementation, tests, and active docs back into sync.

### Task Tracking Discipline

- If a todo list, task list, checklist, plan, or other active tracking surface is in use, update it when each task is completed.
- Do not batch all task-list updates at the very end if task-by-task updates are practical.
- If `task_plan.md`, `progress.md`, or `findings.md` are active, keep them aligned with actual task status.

### Workflow Completion and Archive Check

- After a full `superpowers` workflow cycle or another clearly bounded implementation cycle, check whether the active docs and tracking surfaces suggest that a workstream may be complete.
- If the workspace is NHK-managed and the workstream looks complete, ask whether `nhk-archive` should be invoked.
- Do not archive automatically.

## Documentation Governance and Context Loading

- `documentation-governance.md`, when present, is the source of truth for local documentation lifecycle rules.
- Prefer current code, current tests, and active docs first; treat archive as reference material rather than a default execution source.
- Start with the smallest active document set that can route the task safely.
- Load specs, plans, codemaps, or archive only when the task genuinely needs them.
- For root tracking discipline, use `planning-with-files-zh` when the work is multi-session or needs durable recovery.
- For active specs, plans, TDD, review loops, and implementation workflows, use the relevant `superpowers` skills.

## Subagent Delegation Discipline

- For complex work, prefer decomposing implementation into bounded tasks and dispatching subagents when the subtasks are independent enough.
- Subagent dispatch must follow `subagent-driven-development`; do not improvise a parallel workflow outside that discipline when the task has already been decomposed.
- Dispatch instructions must state whether the subagent is read-only review or write-authorized implementation.
- Give each subagent a clean task brief, owned files or modules, verification expectations, and forbidden actions.
- Do not close a subagent just because a wait timed out.
- Before replacing or closing a subagent, confirm its actual status, current progress, latest conclusion, and whether keeping it alive still reduces risk or rework.

## Implementation Packet Discipline

- Decompose implementation work into bounded packets before dispatch.
- Prefer one primary objective, one main module or surface area, and one verification path per packet.
- Each packet should declare its user-facing goal, owned files, default verification command, and whether it is safe to run in parallel.
- If two packets share the same primary production file or test file, default to serial execution unless the plan explains why parallel work is still safe.
- If a packet grows across unrelated concerns, long execution chains, or multiple verification paths, split it again.

## Blocker Protocol

When blocked during implementation:

1. Field missing producer -> add the producer.
2. Output has no downstream entry -> add to the nearest shared contract.
3. Code forced to understand semantics -> convert to a staged responsibility boundary.
4. Naming conflict -> prefer the current SOS canonical name, then sync code, tests, docs, and plans.

Stop and wait for the human if the fix would change public CLI semantics, pack
merge authority, hidden-library write behavior, source deletion behavior, or the
human-confirmation control model.

## Testing and Verification

- For feature or bugfix work, use test-driven development: write or update tests first, confirm failure when practical, implement, then confirm green.
- For documentation-only changes, run reference checks such as `git diff --check`, file existence checks, and privacy scans for public content.
- For code changes, run the relevant targeted tests first, then the full test suite before calling work complete.
- Default full verification is:

```bash
python -m pytest
```

- CLI smoke check:

```bash
python -m sos --version
```

## Git and Delivery

- Commit format should use conventional prefixes such as `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, or `chore:`.
- Public commits must not include ignored local planning files, local absolute paths, private notes, generated runtime state, caches, or virtual environments.
- Before public commits, check:

```bash
git status --short --ignored
git diff --check
```

- If a sensitive file was accidentally tracked, stop and remove it from the index before pushing.
- Human review is required before broad merges, public behavior changes, or irreversible write-path changes.

---
> Source: [Rainnystone/skill-orchestration-system](https://github.com/Rainnystone/skill-orchestration-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->

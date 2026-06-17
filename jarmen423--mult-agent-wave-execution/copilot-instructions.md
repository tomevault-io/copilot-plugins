## mult-agent-wave-execution

> Use when planning or executing multi-wave parallel work in any repo. Covers wave locking, task ownership, handoff rules, merge gates, and safe subagent splitting by write scope.


# Wave Execution Framework

Use this skill when the work is large enough to justify structured multi-agent execution.

## Read Order

1. `README.md`
2. `docs/quickstart.md`
3. `docs/principles.md`
4. `docs/subagent-splitting.md`
5. `docs/adoption-guide.md`
6. `templates/task-template.md`
7. `templates/handoff-template.md`

## Core Rule

Parallelize by **disjoint write scope**, not by broad topic.

## Workflow

### 1. Lock the wave

Before spawning:

- define the wave goal
- define task ids
- define dependencies
- define write ownership
- define verification commands
- define handoff paths

### 2. Split only non-overlapping work

Safe:

- one task owns `apps/web/**`
- one task owns `packages/core/**`
- one task owns `examples/pack-a/**`

Unsafe:

- two tasks edit the same file
- two tasks edit the same narrow module boundary
- downstream work starts before the shared contract is stable

### 3. Make each task self-contained

Each task should include:

- precise goal
- owned paths
- dependencies
- required verification
- required handoff path

### 4. Require a handoff before closure

Each completed task should leave behind a durable handoff that captures:

- what changed
- what was verified
- blockers or residual risks
- what the next thread should know

### 5. Merge only through gates

Do not call a wave complete until:

- required handoffs exist
- integration review is complete
- verification passes

## Subagent Rules

Use subagents when:

- there are 2 or more independent tracks
- write scopes do not overlap
- the next local step does not block on the delegated result

Keep work local when:

- the task is on the critical path
- the contract is still moving
- ownership is shared or ambiguous

## Operating Note

This skill is intentionally file-first. If a team later adds a database, UI, or orchestration service, those should extend the same wave, task, ownership, handoff, and merge-gate model rather than replacing it.

---
> Source: [jarmen423/mult-agent-wave-execution](https://github.com/jarmen423/mult-agent-wave-execution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

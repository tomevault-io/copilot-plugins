## relatedworksagent

> This repository uses persistent project instructions for coding agents.

# AGENTS.md

This repository uses persistent project instructions for coding agents.

## Planned Development Completion Rule

After completing any implementation step that follows an agreed optimization plan, always do both before ending the task:

1. Update `PROGRESS.md` with the completed work, verification results, and next-step status.
2. Git commit the completed code changes using the existing project commit style, such as `feat(phase/step): 描述`, `fix: 描述`, or `docs: 描述`.

Do not stage or revert unrelated user changes when making that commit.

## Luna Worker Delegation Rule

For complex tasks, the primary agent should delegate suitable subtasks to the
`luna_worker` custom agent when doing so improves development speed or
verification quality.

Delegate only when a subtask is:

- Clearly bounded with an explicit objective and deliverable.
- Independently executable and verifiable.
- Limited to explicitly assigned files or responsibility boundaries.
- Unlikely to conflict with work being performed by another agent.

Good delegation candidates include:

- Focused codebase investigation.
- Implementing an isolated module or a non-overlapping group of files.
- Adding or extending focused tests.
- Reviewing an implementation for bugs, regressions, or missing coverage.
- Running independent verification or analyzing test failures.

Do not delegate:

- Trivial work where coordination costs exceed the benefit.
- Ambiguous architectural or product decisions.
- Work that overlaps files currently being edited by another agent.
- Final integration, repository-wide verification, or release decisions.

Every delegated prompt must state:

1. The exact objective.
2. Owned files or responsibility boundaries.
3. Required verification.
4. Expected report format.
5. That other agents may be editing the repository and their changes must not
   be reverted.
6. That the worker must not commit or update `PROGRESS.md` unless explicitly
   assigned that responsibility.

The primary agent remains responsible for:

- Reviewing and integrating delegated work.
- Resolving cross-module decisions.
- Running final relevant verification.
- Updating `PROGRESS.md`.
- Creating the final Git commit required by this repository.

---
> Source: [lylelllll/RelatedWorksAgent](https://github.com/lylelllll/RelatedWorksAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->

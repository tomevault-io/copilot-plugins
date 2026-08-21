## gofitsv3

> Work efficiently in this repository with minimal token usage.

# AGENTS.md

## Purpose
Work efficiently in this repository with minimal token usage.
Prefer narrow, deliberate changes over broad exploration.

## Operating rules
- Do not scan the whole repository unless it is necessary.
- First identify the smallest set of files relevant to the task.
- For non-trivial tasks, explore first, then briefly summarize the plan before editing.
- Prefer the smallest change that solves the problem.
- Preserve existing architecture and naming unless there is a clear reason to change them.
- Do not perform broad refactors unless explicitly requested.

## Context discipline
- Keep responses concise and focused on the current task.
- Avoid repeating prior analysis.
- Avoid re-reading unchanged files unless needed to verify behavior.
- When searching, prefer targeted search patterns over opening many files.
- If more context is needed, inspect one additional area at a time.
- Summarize long outputs instead of pasting them in full.
- For logs and test output, surface only the relevant failures, errors, and key lines.

## Code changes
- Change only the files needed for the task.
- Keep functions and diffs as small as practical.
- Do not introduce new dependencies unless they provide clear value.
- Prefer standard library and existing project utilities over adding packages.
- Follow the existing style of the repository.
- Avoid placeholder implementations unless explicitly requested.

## Go guidance
- Prefer clear, idiomatic Go.
- Keep error handling explicit.
- Avoid clever abstractions when simple code is sufficient.
- Add or update tests when behavior changes.
- Do not expand scope from a bug fix into cleanup unless asked.

## Frontend guidance
- Preserve existing UI patterns and component structure.
- Do not restyle unrelated areas.
- Keep state changes localized.
- Avoid large framework-level changes unless explicitly requested.
- Do not run expensive image processing on the main UI thread; use a background goroutine with a progress dialog and apply UI updates via the UI thread.

## Testing and validation
- Follow `docs/unit-test-standards.md` as the repository definition of a good unit test and when deciding which code needs unit coverage.
- Use `docs/unit-test-audit.md` as the living inventory of current test quality, missing coverage, and next high-value unit test additions.
- Run the narrowest relevant test, build, or lint command first.
- Do not run full-repo test suites unless the change warrants it or the user asks.
- If a narrow validation passes, report that first.
- If broader validation is needed, state why.
- Report commands run and results briefly.

## Git and safety
- Do not create commits, branches, or PR text unless requested.
- Do not modify secrets, credentials, CI, deployment, or infrastructure files unless the task requires it.
- Do not run destructive commands without explicit user approval.
- Flag risky assumptions before taking risky actions.

## Communication
- Be direct and brief.
- For non-trivial tasks, explore first, then plan, then edit.
- When starting work, state:
  1. relevant files
  2. intended change
  3. validation plan
- When finishing, state:
  1. files changed
  2. what changed
  3. validation performed
  4. any remaining risk, assumptions, or follow-up

## Cost control
- Optimize for minimal context use.
- Prefer targeted reads, targeted edits, and targeted validation.
- Avoid broad repo scans unless clearly necessary.
- If one or two nearby files must be inspected to verify correctness, do that and explain why.
- Ask before expanding into refactors, unrelated cleanup, or architecture changes.
- If a workflow becomes repetitive or long, suggest creating a skill or subagent instead of enlarging this file.

---
> Source: [wierdling/gofitsv3](https://github.com/wierdling/gofitsv3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

## challenge-repo-setup

> AI Challenge repository operating rules and defaults


# AI Challenge 2026 - Cursor Rule

## Repository Boundaries

- This repository contains four isolated task workspaces: `task-1`, `task-2`, `task-3`, `task-4`.
- Never initialize an application at the repository root.
- Never create root-level `package.json`, lockfiles, or root app scaffolding.
- When starting implementation for a task, initialize tooling only inside that task's directory.

## Baseline Technology Defaults

Use these defaults unless a specific task announcement says otherwise:

- Node.js 24
- React latest stable
- React Bootstrap for UI components
- Font Awesome for icons

## Task Execution Protocol

1. Read the task announcement completely.
2. Convert requirements into an explicit checklist.
3. Ask clarifying questions immediately when requirements are ambiguous.
4. Implement with small, verifiable steps.
5. Re-check each requirement before final submission.

## Quality and Submission Discipline

- Prioritize correctness over speed.
- Validate all required deliverables and artifacts.
- Verify that links and deployed resources are reachable and correct.
- Remove incomplete placeholders and unresolved TODO items before finalizing.
- Assume one-shot submission quality expectations for bonus eligibility.

## Required Task Documentation

Maintain/update `task-X/README.md` with:

- task objective and scope
- implementation summary
- setup/run instructions
- verification checklist
- submission links and artifacts

## Agent Behavior Requirements

- Restate acceptance criteria before implementation.
- Surface assumptions and risks explicitly.
- Ask questions instead of guessing when uncertainty impacts correctness.
- Provide a concise verification report when work is complete.
- Avoid destructive operations unless explicitly requested.

## Safety and Repo Hygiene

- Never commit secrets, tokens, or credentials.
- Keep naming clear and consistent.
- Prefer deterministic commands and reproducible instructions.
- Keep changes limited to files relevant to the active task.

---
> Source: [fedar-piashko-vention/edu-ai-challenge-2026](https://github.com/fedar-piashko-vention/edu-ai-challenge-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->

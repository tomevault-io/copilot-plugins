## project-context

> Project context and memory for the CYA Playground benchmark project


# CYA Playground - AI Agent Context

The README.md has the full project description, results, and PR links.
This file contains agent-specific working notes only.

## Branch Naming
Always prefix with `mikec/`, e.g. `mikec/feature-user-profiles`

## Tech Stack
- Convex (backend), React 19, Vite, Tailwind CSS v4, Convex Auth (password)
- Schema: users, projects, projectMembers, tasks, comments, activityLog, labels, taskLabels

## PR Expectations Workflow
- Each PR has a `pr-expectations/pr-NN-<slug>.md` file documenting:
  - Branch name and cover story
  - What we are testing (one focused thing per PR)
  - What a tool should / should not say
  - Detailed tool reviews and scores
- Scoring criteria are in `pr-expectations/SCORING.md`

## Decisions Log
- **No unit tests**: Benchmark tests code review intelligence, not runtime correctness.
- **No return validators**: Dropped all `returns:` validators from Convex functions. Args validators still required.

---
> Source: [mikecann/cya-playground](https://github.com/mikecann/cya-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->

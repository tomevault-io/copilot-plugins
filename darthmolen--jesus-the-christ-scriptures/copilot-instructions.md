## jesus-the-christ-scriptures

> - **Project Name**: Scriptures: Jesus the Christ

# Scriptures: Jesus the Christ

## Project Context

- **Project Name**: Scriptures: Jesus the Christ
- **Primary Language**: C#
- **Framework**: .NET 10 Maui
- **Repository**: jesus-the-christ-scriptures

## SKILLS

- csharp-quality-developer
- /using-superpowers: at any time, whenever the user wants to start planning, discussing code, etc.
- /super-powers:brainstorm: whenever the user wants to implement or discuss plans or any deep discussions
- /super-powers:execute-plan: whenever a plan needs to be started or executed

## Planning Protocol

### Directory Structure

Maintain these directories for planning artifacts:

```
planning/
├── backlog/            # Future work items and deferred tasks
├── in_progress/       # Active phase documents
└── completed/         # Finished phase documents with outcomes
```

### Pre-Task Requirements

Before executing any task:

1. Create a phase document in `planning/in_progress/` named `phase_[name]_[date].md`
2. Document the objective, approach, and success criteria
3. List all files expected to be modified
4. Identify dependencies and prerequisites

### Post-Task Requirements

After completing any task:

1. Update the phase document with actual outcomes
2. Move completed phase documents to `planning/completed/`
3. Document any deviations from the plan
4. Note lessons learned or issues encountered
5. **COMMIT**: All work must be committed with a descriptive message before moving to the next phase

## Commit Protocol

### Mandatory Commit Points

- After completing each phase
- Before switching to a different task
- After any significant refactoring
- Before planning mode discussions about architectural changes

### Commit Message Format

```
[PHASE] Brief description

- Specific change 1
- Specific change 2

Phase: planning/completed/phase_[name]_[date].md
```

## Branch & Merge Policy

**Nothing goes directly to `main`. Ever.** No direct commits or pushes to `main` — not by
the agent, not by tooling, no exceptions.

- Do all work on a feature branch (e.g. `feature/<name>` or `claude/<name>`).
- Open a pull request into `main`.
- **Only the repository owner (`darthmolen`) approves and merges.** The agent must not merge to
  `main`; it stops at "PR opened" and reports the PR URL.
- `main` is protected on GitHub (PR required, enforced on admins, no force-push/deletion). If a
  push to `main` is rejected, that is expected — branch and open a PR instead.
- The agent may push **feature branches** and open PRs freely; it may not merge them.

---
> Source: [darthmolen/jesus-the-christ-scriptures](https://github.com/darthmolen/jesus-the-christ-scriptures) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

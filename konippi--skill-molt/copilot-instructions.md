## skill-molt

> Use after completing a task that involved significant trial-and-error, unexpected behavior, or non-obvious workarounds. Use when the user says 'what did we learn', 'improve skills', or 'shrink skills'. Also use when a skill feels outdated, bloated, or leads to wrong outcomes. Do NOT use for straightforward tasks, simple bug fixes, or when the session went smoothly without surprises. Do NOT use when the user asks about skill-molt itself (installation, configuration, usage).


# Skill Molt

Create and improve agent skills by reflecting on sessions. Extract only what you could NOT have known before this session.

## Core Principles

1. **Non-inferable only**: If you could discover it by reading the codebase, do not put it in a skill. Only include tooling gotchas, non-obvious conventions, and lessons from failure.
2. **Procedures, not documentation**: Write step-by-step actions. Never explain what something is — show what to do.
3. **Human decides**: Generate or update SKILL.md files but do NOT save without user approval. Present changes for review.

## Routing

Determine which phase to execute:

- **"What did we learn?"** or session involved trial-and-error, error recovery, or non-obvious workarounds → Read `references/observe-phase.md`, then `references/generate-phase.md`
- **"Improve skills"** or existing skill feels outdated → Read `references/observe-phase.md`, then `references/improve-phase.md`
- **"Shrink skills"** or skill feels bloated or agent follows a skill instruction that leads to a wrong outcome → Read `references/decay-phase.md`

After generating, improving, or applying decay, always run `references/validate-phase.md` before presenting to the user.

## When NOT to Create a Skill

Stop if ANY apply:

- The task was a one-off with no reusable pattern
- The knowledge is discoverable from the codebase (README, config files, code comments — you do NOT need to read every file; check the obvious sources first)
- An existing skill already covers 80%+ of the use case — update it instead
- The lesson is specific to one file, not a recurring pattern

## Workflow Overview

```text
Observe → Generate or Improve → Validate → Present to user
Decay → Validate → Present to user
```

Each phase has detailed instructions in `references/`. Load them only when you reach that phase.

---
> Source: [konippi/skill-molt](https://github.com/konippi/skill-molt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

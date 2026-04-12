## openshift-lp-qe-tools

> Upon first interaction in this directory:

# Project Instructions
## Mandatory First Action
Upon first interaction in this directory:
 1. Check if `.AI_README.md` exists. If it does not exist, create it using the detailed templates below.
 2. Check if `.AI_HISTORY.md` exists. If it does not exist, create it with the history template below.

## .AI_README.md
### Root Directory Template
```markdown
# AI Agent Instructions
## Project Overview
[Brief description of what this project contains, its purpose, and main technologies used.]

## Key Constraints
[Any limitations, requirements, or rules to follow - e.g., coding standards, dependencies to avoid, performance requirements.]
- **Best Practices**: Re-read https://edttjredhat.github.io/RHP-Wiki/Confluence/MPEXIENG/Test+Infrastructure/Best+Practises/Section0.html at:
  - Start of new sessions.
  - Session resume (when continuing from a previous conversation).
  - Before writing NEW bash scripts or functions (not minor edits to existing code).
  - When user explicitly requests best practices review.

## Architecture Notes
[Important structural or design decisions - e.g., folder structure, design patterns, data flow.]

## Common Tasks
[Frequent operations or workflows - e.g., how to build, test, deploy, or common debugging steps.]

## Dependencies and Setup
[Key dependencies, environment variables, setup instructions.]

## References
[Links to relevant documentation, APIs, or resources.]
```

### Sub-Directories Template (Optional)
For sub-directories with significantly different context, create a simpler `.AI_README.md` using this template:
```markdown
# AI Agent Instructions
## Purpose
[What this subdirectory contains and why it exists.]

## Local Conventions
[Any patterns, rules, or approaches specific to this area.]

## Notes
[Important context that differs from root]
```

Use subdirectory AI.md files when:
  - Different tech stacks are used.
  - Different teams own the code.
  - Domain-specific knowledge is required.
  - Conventions differ significantly from root.
  - Explicitly desired for modular approach.


## .AI_HISTORY.md
### Template
```markdown
# AI History Log
This file tracks important changes and decisions in this directory and its sub-directories (when sensible).

## YYYY-MM-DD
### Initial Setup
- Project initialized
```

### Maintaining .AI_HISTORY.md
**Automatically update `.AI_HISTORY.md` immediately after each qualifying change, without being asked.** Do not defer to end of session (sessions may span
weeks or crash unexpectedly). Entries are grouped by date (one `##` per calendar day, sorted newest-first). For each qualifying change, add a new `###` entry.
If today's `## YYYY-MM-DD` already exists, append the `###` to it (no empty line between `###` sections, chronological order within the day), otherwise prepend
a new `## YYYY-MM-DD` as the first `##` section.

Add an entry whenever any of the following occur:
  - Significant architectural changes are made.
  - Major refactoring occurs.
  - Important dependencies are added/removed/updated.
  - Critical bugs are fixed.
  - New patterns or conventions are established.
  - Breaking changes are introduced.
  - Major features are added.

Entry format:
```markdown
## YYYY-MM-DD
### Brief Title 1
- Bullet point description of change.
- Why the change was made (if relevant).
- Any important context or decisions.
### Brief Title 2
- ...
 .
 .
 .
### Brief Title N
- ...
```

Scope: Each `.AI_HISTORY.md` tracks changes in its directory and sub-directories. For changes that also involve sub-directories with their own
`.AI_HISTORY.md`, record at the level that best fits the context of the change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RedHatQE)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RedHatQE)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## kavach

> This file is the entry point for any AI agent working on this repo.

# Project Name — Agent Instructions

This file is the entry point for any AI agent working on this repo.

## MANDATORY: Load Workflow First

Before doing any work, read these files in order:
1. **`agents/workflow.md`** — methodology and pipeline
2. **`agents/memory.md`** — project knowledge, principles, tooling
3. **`agents/journal.md`** (last ~50 lines) — recent progress
4. **`docs/plans/README.md`** — check for active plan to resume

These files govern how you work. Do not skip them.

---

## Project Overview

<!-- Replace with a 2-3 sentence description of what this project does. -->

**Project Name** is ...

## Repository Structure

```
project-root/
  CLAUDE.md                      <-- You are here
  agents/                        <-- AI operational files
    workflow.md                  <-- Methodology and pipeline (READ FIRST)
    memory.md                   <-- Project knowledge, principles, tooling
    journal.md                  <-- Chronological progress log
  docs/
    requirements/                <-- Per-module: what and why (human audience)
      README.md                 <-- Module summary and references
    design/                     <-- Per-module: how and why (human audience)
      README.md                 <-- Patterns, guidelines, module references
    stories/                    <-- Story specs + backlog dashboard (AI audience)
      README.md                 <-- Prioritized story dashboard
    plans/                      <-- Active plan + archived plans
      README.md                 <-- Current active plan
  solution/                     <-- All source code lives here
```

## Key Design Principles

<!-- Replace with 3-5 principles that guide this project's architecture. -->
<!-- These should also be added to agents/memory.md under Project Principles. -->

- **Principle 1** — ...
- **Principle 2** — ...
- **Principle 3** — ...

## Working with this Repo

### Commands (run from `solution/`)

<!-- Replace with actual commands for your project. -->

```bash
# Tests
npm run test                      # Run all tests

# Lint
npm run lint                      # 0 errors expected
```

**Important:** Always run commands from the `solution/` directory.

## Conventions

### Pipeline for stories
For non-trivial work: **Story → Plan → Implement**.
See `agents/workflow.md` for the full process.

### Two documentation layers
- **Requirements + Design** — human audience, per module, what/why and how/why
- **Stories** — AI audience, actionable specs with acceptance criteria and examples

### Creating a new story
1. Create story file in `docs/stories/NNN-<story>.md` (flat, no subdirectories)
2. If new module: create requirements and design docs first
3. Add to `docs/stories/README.md`
4. Follow the pipeline in `agents/workflow.md`

### When completing work
1. Run tests and lint — both must pass
2. Check requirements and design docs are still accurate
3. Archive plan to `docs/plans/<datetime>-<story>.md`
4. Update `agents/journal.md` with summary and commit hashes
5. Mark story done in `docs/stories/README.md`

### Handling interrupts
All interrupts go to `docs/stories/README.md`. Pick up next.

### Lint Rules
- Warnings are pre-existing and acceptable
- **Errors must be zero**

## Key Files Quick Reference

| File | Purpose |
|------|---------|
| `agents/workflow.md` | Methodology, pipeline, session lifecycle |
| `agents/memory.md` | Project knowledge, principles, tooling |
| `agents/journal.md` | Chronological progress log |
| `docs/requirements/` | Module requirements — what and why |
| `docs/design/` | Module design — how and why, patterns |
| `docs/stories/` | Story specs (flat, NNN-name.md) |
| `docs/stories/README.md` | Prioritized story dashboard |
| `docs/plans/README.md` | Current active plan |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerrythomas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

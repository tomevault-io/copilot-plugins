## solar-sim

> This file provides instructions for Claude Code agents working on Solar-Sim.

# CLAUDE.md - Agent Instructions

This file provides instructions for Claude Code agents working on Solar-Sim.

## Quick Start

If you're a Claude agent starting fresh, read this section first.

### Project Overview

Solar-Sim is a webapp for calculating sun hours and light categories for any location on Earth. But before implementing the app itself, we're building the **multi-agent development infrastructure** that enables concurrent autonomous development.

### Key Files to Read

1. `docs/specification.md` - Technical specification (living document)
2. `docs/happy_path.md` - Target user experience
3. `docs/active/ROADMAP.md` - Current progress and priorities
4. `task-graph.yaml` - DAG of tasks (what to work on)

### Current Phase

**Phase: Foundation & Tooling**

We are implementing the multi-agent workflow tooling before the application itself:
- Ralph Loop integration (`just ralph`)
- DAG parsing and prompt generation (`just prompt`)
- Git worktree management for parallel development

---

## Workflow Commands

Use `just` to interact with the project:

```bash
# See all available commands
just

# Core workflow
just ralph          # Start autonomous coding loop (processes tasks until done)
just ralph-status   # Check loop heartbeat and recent activity
just ralph-logs     # Tail logs in real-time with formatted output
just prompt         # Get next task prompt from DAG
just dag-status     # View task graph status
just dag-refresh    # Regenerate DAG from doc frontmatter

# Worktrees for parallel development
just worktree-new <name>    # Create new worktree
just worktree-list          # List active worktrees
just worktree-remove <name> # Remove a worktree

# Development (once SvelteKit is initialized)
just install        # Install dependencies
just dev            # Start dev server
just build          # Production build
just test           # Run tests
```

---

## How to Pick Up Work

### 1. Check the Task Graph

```bash
just dag-status
```

Or read `task-graph.yaml` directly. Look for tasks with `status: ready` (dependencies met, available for work).

### 2. Understand the Task

Each task references:
- A **story** in `docs/active/stories/S-NNN-*.md` (feature context)
- Possibly **tickets** in `docs/active/tickets/` (atomic subtasks)
- Related **research** in `docs/knowledge/research/`
- Relevant **requirements** in `docs/knowledge/requirements/`

Read these before implementing.

### 3. Claim and Execute

1. Update `task-graph.yaml` to set your task to `status: in-progress`
2. Implement the task
3. Update status to `complete` when done
4. Update `docs/active/ROADMAP.md` with progress

### 4. Research-Plan-Implement Pattern

For non-trivial work, follow this pattern:

1. **Research**: Gather information, evaluate options, document findings in `docs/knowledge/research/`
2. **Plan**: Create or update stories/tickets with implementation approach
3. **Implement**: Write code, following the plan

---

## Writing Style

All documentation must be written for speed reading. This is a hard requirement. Read `docs/knowledge/requirements/writing-style.md` for the full guide, but here's the essence.

Write as if you're explaining something to a colleague out loud. Use flowing prose with complete sentences that build on each other. Each word should lead naturally to the next.

Use active voice and transitions like "because," "so," "which means," and "then" to create forward momentum. Lead with context by telling the reader why before telling them what.

Avoid bullet lists, tables, headers-as-content, and telegraphic style. These visual structures lose their meaning when read word-by-word. Instead of listing items as bullets, weave them into sentences. Instead of a procedures list, write "start by doing X, then do Y, and finally Z."

When in doubt, read your writing out loud. If it sounds choppy or unnatural, rewrite it.

Code blocks are fine since readers expect to parse them differently. YAML frontmatter and structured data files are also exempt since they're for machines.

---

## Document Conventions

Stories live in `docs/active/stories/` with names like `S-NNN-title.md`. They describe features at a high level including context, motivation, acceptance criteria, and related research. Tickets live in `docs/active/tickets/` and represent atomic implementable tasks with clear scope and dependencies.

All story and ticket documents should include YAML frontmatter for DAG parsing. The frontmatter specifies the id, title, status, priority, and dependencies. The `just dag-refresh` command will eventually parse this frontmatter to regenerate the task graph.

---

## Git Workflow

The main branch should always be in a stable, working state. Feature branches follow the pattern `feature/*` with typically one branch per story, and research spikes that may not merge use `research/*`.

Multiple agents can work simultaneously using Git worktrees. Run `just worktree-new ralph` to create a worktree at `../solar-sim-ralph` on a `feature/ralph` branch, then cd into it and work independently. Each worktree is an independent working directory that shares Git history with the main repo.

Commits follow the conventional format with a type, scope, and description. Types include feat, fix, docs, refactor, test, chore, and research. For example, `feat(solar): add sun position calculation module` or `docs(spec): update architecture diagram`.

### Cross-Worktree Coordination

The main worktree owns the authoritative copy of task-graph.yaml, so agents in linked worktrees should not modify their local copies directly. Instead, task status updates should be included as part of the PR that gets merged back to main.

When an agent in a linked worktree wants to pick up new work, they should first verify task availability by fetching the latest main branch with `git fetch origin main` and then reading the current DAG state with `git show main:task-graph.yaml`. This ensures they see which tasks are actually available rather than relying on potentially stale local data. The `just prompt` command will print a warning when running from a linked worktree as a reminder to check availability on main.

After completing work, the agent pushes their branch with `git push -u origin feature/<name>` and creates a PR using `gh pr create`. The PR should include the task status update that marks the task complete. When the PR merges, that merge serves as the serialization point for task claiming. If two agents happen to claim the same task, the second agent's PR will show a merge conflict on task-graph.yaml, making the duplication visible so the second agent can pick a different task instead.

### Worktree Workflow Example

The complete cycle from creating a worktree through cleanup looks like this. Start by creating a worktree from the main repo with `just worktree-new alpha`, which creates a new worktree at `../solar-sim-alpha` with a `feature/alpha` branch. Then change into that directory with `cd ../solar-sim-alpha`.

Before starting work, check task availability by running `git fetch origin main` followed by `git show main:task-graph.yaml | grep -A5 "status: ready"` to see which tasks are actually available. Pick an unclaimed task and do the implementation work, committing changes as you go with `git add -A && git commit -m "feat(scope): description"`.

When the work is complete, push the branch with `git push -u origin feature/alpha` and create a PR with `gh pr create --title "T-XXX-YY: Brief description" --body "Summary of changes"`. The PR body should describe what changed and include a test plan.

After the PR is reviewed and merged, return to the main worktree with `cd ../solar-sim` and clean up by running `just worktree-remove alpha`. The branch will be deleted automatically since it's been merged.

---

## Code Standards

TypeScript should use strict mode with explicit return types on exported functions, preferring interface over type for object shapes. Svelte components should use TypeScript via `<script lang="ts">`, prefer composition over inheritance, and stay focused and small.

Testing strategy includes unit tests for calculation modules, integration tests for API routes, and end-to-end tests for critical user flows once the UI exists.

---

## Current Priorities

Check `docs/active/ROADMAP.md` for the latest priorities. The foundation phase focuses on S-002 for DAG parsing and prompt generation since that's the critical path, followed by S-003 for git worktrees and S-001 for the Ralph loop as a stretch goal. Once the workflow tooling is in place, S-004 initializes the SvelteKit project and application features can begin.

---

## Troubleshooting

If you see "no package.json found," that's because SvelteKit hasn't been initialized yet, which is tracked in the S-004 story. If "Ralph Loop not yet implemented" appears, the workflow tooling is still being built so check the story status in task-graph.yaml.

Merge conflicts between worktrees shouldn't happen if each worktree works on a separate branch. Coordinate through clear task assignment in task-graph.yaml, non-overlapping file ownership per story, and regular rebasing onto main.

---

## Contact

This is an autonomous multi-agent project where coordination happens through task-graph.yaml for work assignment, docs/active/ROADMAP.md for progress tracking, and Git commit history as the work record. No human in the loop is required for routine operation, but humans can intervene by updating docs or the task graph.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnhkchen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/johnhkchen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

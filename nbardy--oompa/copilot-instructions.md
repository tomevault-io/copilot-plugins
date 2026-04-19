## oompa

> **This is critical - don't mix them up.**

# Oompa Loompas - Project Conventions

## docs/ vs notes/

**This is critical - don't mix them up.**

### `docs/` - Living Documentation
- **Always up to date** with the current state of the project
- README, API reference, configuration guides, getting started
- If the code changes, docs/ must be updated to match
- These are the "source of truth" for how things work NOW

### `notes/` - Static Exploration Files
- **Frozen in time** - written once, rarely updated
- Planning documents, design explorations, thinking out loud
- Blog posts, essays, comparisons written at a specific point
- Leave a trail of how we got here - the "archaeological record"
- Naming convention: `YYYY-MM-DD__topic__description.md` for dated notes

### Quick Reference

| Folder | Updates? | Purpose | Examples |
|--------|----------|---------|----------|
| `docs/` | Yes, kept current | How it works now | README, API, Config |
| `notes/` | No, static | How we got here | Explorations, designs, essays |

### When Adding Files

**Ask yourself:** "Will this need updates when the code changes?"
- Yes → `docs/`
- No → `notes/`

## Agent Notes Convention

The `agent_notes/` folder is for agent-to-agent communication during swarm execution:
- `scratch/` - Working notes, can be deleted
- `ready_for_review/` - Completed work awaiting review
- `notes_FROM_CTO/` - Feedback from reviewer agents
- `proposed_tasks/` - Self-created tasks from workers

## Systems Design

See [docs/SYSTEMS_DESIGN.md](docs/SYSTEMS_DESIGN.md) for the constitution governing agent vs. framework responsibilities. Key rule: **agents act, the framework reacts.** Agents get maximum flexibility (claim, create, work); deterministic software handles races, merges, completion, and stability. All design decisions should reference this document.

## Git Workflow

- Workers operate in isolated git worktrees
- Everything parallel except the "rebase + merge" moment
- Only 2 agent roles: PROPOSER and REVIEWER
- Workers can create their own tasks (self-directed)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nbardy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

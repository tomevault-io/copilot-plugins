## auto-dev-agent-template

> Each agent sticks to its lane and updates labels/status.

# Agents (KISS)

Each agent sticks to its lane and updates labels/status.

## Product Owner (PO)
- Owns Epics/Stories (issue forms).
- Writes acceptance criteria.
- Prioritizes backlog; moves selected stories to `status:todo` and the current Milestone (Sprint).

## Scrum Master
- Runs sprints (Milestones).
- Keeps board clean: `status:*` transitions.
- Splits stories into `type:task`.

## Dev
- Works only on `type:task`.
- Uses one small PR per task. Links PR to issue.
- Updates status: `todo → doing → review → done`.

## QA
- Files `type:bug`, validates acceptance criteria, approves PRs if criteria pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chichha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chichha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

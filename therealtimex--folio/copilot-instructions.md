## folio

> This project uses an "Autonomous Engineer" workflow driven by GitHub Issues.

# Gemini Agent Context - Folio

This project uses an "Autonomous Engineer" workflow driven by GitHub Issues.

## Task Picking Protocol
- **Label:** `agent-task`
- **Pick-up Command:** `gh issue list --label "agent-task"`
- **Execution:** When you pick up a task, you should:
  1.  Create a branch for the issue: `gh issue develop <issue-number>` (or manual `git checkout -b task/<issue-number>-...`).
  2.  Implement the feature/fix.
  3.  Verify with tests.
  4.  Close the issue and delete the branch.

## Creating Tasks
Anyone (human or agent) can create a task via:
```bash
npm run task "Task Title" "Task Description"
```
This automatically tags the issue with `agent-task` for visibility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/therealtimex)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/therealtimex)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

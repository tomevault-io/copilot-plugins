## gemini-ralph-loop

> Ralph Loop enables iterative, self-referential development where you work on a task repeatedly until completion. Your work persists between iterations.

# Ralph Loop - Autonomous Development Loop

Ralph Loop enables iterative, self-referential development where you work on a task repeatedly until completion. Your work persists between iterations.

## State Management

Ralph Loop stores state in `.ralph-state/` directory:

- `.ralph-state/state.json` - Current loop state
- `.ralph-state/history.json` - Iteration history
- `.ralph-state/checkpoints/` - Saved checkpoints

## State File Format

`.ralph-state/state.json`:

```
{"id": "loop-
```

## Commands

/ralph:start-loop `<task>` - Start a new loop
/ralph:status - Check current status
/ralph:pause - Pause the loop
/ralph:resume - Resume paused loop
/ralph:complete - Mark as completed
/ralph:cancel - Cancel the loop
/ralph:history - View iteration history
/ralph:checkpoint `<name>` - Create checkpoint
/ralph:restore `<name>` - Restore checkpoint
/ralph:reset - Clear all state
/ralph:help - Show help

## Workflow

Start: /ralph:start-loop "Build a REST API"
Work iteratively, updating .ralph-state/state.json after each step
When done, output: `<done>`COMPLETE `</done>`

## Important Rules

Always read .ralph-state/state.json before making changes
Increment iteration counter after each meaningful progress
Files you create persist between iterations
Use checkpoints before risky changes

---
> Source: [kranthik123/Gemini-Ralph-Loop](https://github.com/kranthik123/Gemini-Ralph-Loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->

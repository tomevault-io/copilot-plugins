## agent-loop

> General-purpose task execution loop using GitHub Issues as the task queue.

# AGENTS.md

General-purpose task execution loop using GitHub Issues as the task queue.

## How It Works

1. `prompt.md` defines the workflow (fetch → plan → execute → verify → commit)
2. GitHub Issues contain the actual tasks/requirements
3. `loop.sh` runs droid repeatedly until issues are done

## Key Files

- `prompt.md` - Edit this to customize the workflow
- `loop.sh` - The loop runner
- `progress.txt` - Running log

## Running

```bash
./loop.sh 10      # Run 10 iterations
./loop.sh 1       # Single task
```

## Customizing prompt.md

The prompt defines HOW to work, not WHAT to build. Edit:
- **Step 5** - Add your feedback loops (test commands)
- **Step 2** - Change priority rules
- **Step 7** - Adjust commit conventions

## The Issue Is The Spec

Write detailed issues. The issue body is the only context droid has about what to build.

---
> Source: [bentossell/agent-loop](https://github.com/bentossell/agent-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->

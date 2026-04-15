## airswipe

> description: "General project rules: maintain logs, memory, lessons learned, and scratchpad"

# AirSwipe Project Rules

description: "General project rules: maintain logs, memory, lessons learned, and scratchpad"
globs: "**/*"
alwaysApply: true

---

## Logging and Memory Rules

### Start of Each Task

Before performing any work:

1. Read the existing context files in the `.cursor` folder:
   - `progress.md` (progress log)
   - `activeContext.md` (recent context)
   - `lessons-learned.md`
   - `scratchpad.md`

2. Summarize what you read to understand the project's current state, outstanding tasks, and previous issues.

### Log Every Task

After completing each discrete task, append a new entry to `.cursor/progress.md` with:

- **Timestamp** - Current date/time
- **Task** - Brief description of what you attempted or completed
- **Reason** - Why you made this change or decision
- **Changes Made** - Files and functions edited plus a short summary
- **Commands Run** - Any terminal commands, tests, or scripts executed
- **Errors Encountered** - Error messages or problems faced, even if resolved

Always append at the end of the file without overwriting existing entries.

### Update Lessons and Scratchpad

- When you discover a new insight, pitfall, or decision that others should know, append a concise note to `.cursor/lessons-learned.md`

- Use `.cursor/scratchpad.md` to track tasks and their status:
  - `[ ]` - To-do
  - `[X]` - Done
  - `[!]` - Blocked
  - `[?]` - Question
  - Include dependencies and confidence levels when relevant

### Periodic Checks and Summarization

- If `.cursor/progress.md` or other memory files become very long (>1000 lines), summarize older entries into a high-level bullet list, then archive or delete detailed lines. Summaries should retain key decisions, issues, and outcomes.

- Before starting a complex task or when errors occur, review `progress.md`, `lessons-learned.md`, and `scratchpad.md` to recall what has been tried and avoid repeated mistakes.

---

## Coding Standards

- Follow established coding standards, naming conventions, and best practices
- Use structured logging (e.g., `logger.error()`) when writing code
- Avoid hard-coded values when configuration inputs are available
- Break large problems into smaller steps
- Plan changes with reasoning
- Verify work using tests or linting where possible

---

## Project-Specific Notes

- **Stack:** Python, sounddevice, numpy, scipy, matplotlib, (optional) PyTorch
- **Architecture:** Modular design in `airswipe/` package
- **Entry point:** `main.py` with CLI subcommands
- **Design doc:** `NEWDESIGNDOC.md` contains full specification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phillipyan300) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## sign-language-translator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ASL Sign Language Translator — converts text input into American Sign Language (ASL) fingerspelling images using PIL and matplotlib. Each letter is mapped to a pre-cropped hand gesture image and concatenated horizontally to form the visual translation.

# Workflow Orchestration

### 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy to keep main context window clean

- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 2a. Subagent Mandatory Rules

Every subagent prompt MUST include these rules:

- **Follow specs literally:** When an RFC/spec exists, subagents MUST follow it exactly. If a subagent believes a different approach is better, it must NOT implement the deviation — only flag it in its output and explain its reasoning. Wait for human approval before changing the approach.

### 3. Self-Improvement Loop

- After ANY correction from the user: update '.claude/tasks/lessons.md' with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests -> then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to '.claude/tasks/todo.md' with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review to '.claude/tasks/todo.md'
6. **Capture Lessons**: Update '.claude/tasks/lessons.md' after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Commands

```bash
# Set up environment (uses uv)
uv sync

# Translate text to ASL (saves to output/translation.jpg + shows matplotlib window)
uv run translator.py "HELLO WORLD"
uv run translator.py              # interactive prompt mode

```

## Architecture

**Data flow:** Text input → character-by-character lookup → load individual ASL letter images → horizontally concatenate with PIL → save to file + display with matplotlib.

- **`translator.py`** — Core translator. `translate(text, save_path)` loads each letter image from `Assets/single_chunk/`, creates blank images for spaces, concatenates all horizontally with `Image.paste()`, saves result and displays via `plt.show()`.
- **`Assets/Chunk.jpg`** — Source ASL alphabet chart (A-Z hand gestures, by freepik.com).
- **`Assets/single_chunk/`** — Individual letter images (`A.jpg`–`Z.jpg`).
- **`output/`** — Generated translation images (gitignored).

## Dependencies

Managed with `uv`. Python >=3.12. Core deps: Pillow (image processing), matplotlib (display).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SoyAndresTrujillo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SoyAndresTrujillo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

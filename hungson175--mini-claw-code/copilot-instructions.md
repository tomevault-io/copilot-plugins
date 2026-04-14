## mini-claw-code

> Educational project demonstrating the absolute minimum implementation of Claude Code.

# Mini Claw Code

Educational project demonstrating the absolute minimum implementation of Claude Code.

## What This Is

A ~60-line Python script that recreates the core concept of Claude Code using only 2 tools:
1. **bash** - Execute shell commands
2. **todowrite** - Track tasks

The point: Claude Code is just an **agentic loop** (LLM calls tools, gets results, calls more tools, until done). Everything else is optimization.

## Project Structure

```
main.py                  -- The agent loop
prompts/system_prompt.txt -- System prompt
prompts/bash.txt          -- Bash tool description
prompts/todowrite.txt     -- TodoWrite tool description
```

## Running

```bash
uv sync && uv run python main.py
```

Requires `XAI_API_KEY` in `.env` (uses xAI's grok-4-fast-non-reasoning).

## Rules

- Keep this project minimal. Do not add more tools or complexity.
- The entire point is educational simplicity.
- No comments in code unless absolutely necessary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hungson175)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hungson175)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

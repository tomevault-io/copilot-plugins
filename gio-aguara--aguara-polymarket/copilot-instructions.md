## aguara-polymarket

> You are a collaborating ML engineer on this project. Read PROJECT.md for full context.

# Gemini Instructions — Aguara

## Role
You are a collaborating ML engineer on this project. Read PROJECT.md for full context.

## Multi-Agent Protocol
This project uses multiple AI agents (Claude, Gemini, others). Coordination happens through files in `.agents/`.

### Before Starting Work
1. Read `.agents/taskboard.md` — check what's assigned to you and what others are doing
2. Read `.agents/comms.md` (last 20 lines) — check for messages from other agents
3. Read `.agents/decisions.md` — respect architectural decisions already made

### While Working
- Update your task status in `taskboard.md` when you start/finish work
- If you need input from another agent, post to `.agents/comms.md`
- If you make an architectural decision, log it in `.agents/decisions.md`

### Code Reviews
- When asked to review, read the file from `.agents/reviews/` and respond in the same file
- When requesting review, create `.agents/reviews/gemini_review_NNN.md`

### Conventions
- Use Polars (not pandas) for all dataframe operations
- Type hints on all function signatures
- Docstrings on public functions only
- Test files go in `tests/` mirroring `src/` structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gio-aguara)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gio-aguara)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## claude-code-rlm

> This repository includes a minimal "Recursive Language Model" (RLM) setup for Claude Code:

# Project instructions

## RLM mode for long-context tasks

This repository includes a minimal "Recursive Language Model" (RLM) setup for Claude Code:
- Skill: `rlm` in `.claude/skills/rlm/`
- Subagent (sub-LLM): `rlm-subcall` in `.claude/agents/`
- Persistent Python REPL: `.claude/skills/rlm/scripts/rlm_repl.py`

When the user needs you to work over a context that is too large to paste into chat:
1) Ask for (or locate) a context file path.
2) Run the `/rlm` Skill and follow its procedure.

Keep the main conversation light: use the REPL and subagent to do chunk-level work, then synthesise.

---
> Source: [brainqub3/claude_code_RLM](https://github.com/brainqub3/claude_code_RLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->

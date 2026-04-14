## wasp3

> Wasp3 is a pip-installable Python development agent powered by Claude. It mirrors Claude Code's

# Wasp3 — Python Development Agent

## Project Overview
Wasp3 is a pip-installable Python development agent powered by Claude. It mirrors Claude Code's
architecture: persistent memory between sessions, project-aware CLAUDE.md context loading,
reusable SKILLS.md definitions, GitHub integration, and a tool-calling agentic loop.

Primary focus: **Python software development** — writing code, running tests, debugging, and
analysing codebases. kdb+/q integration is available but secondary.

## Architecture
- `src/wasp3/llm/` — Pluggable LLM client abstraction (Anthropic by default)
- `src/wasp3/tools/` — Tool implementations: files, python execution, pytest, git, GitHub
- `src/wasp3/agent.py` — Core agentic loop (tool dispatch + message threading)
- `src/wasp3/context.py` — CLAUDE.md chain loader (home → cwd)
- `src/wasp3/memory.py` — Persistent memory at ~/.wasp3/memory/
- `src/wasp3/skills.py` — SKILLS.md parser and /skill-name expansion
- `src/wasp3/cli.py` — Typer CLI entry point

## Adding a New Tool
1. Implement the function in the appropriate `src/wasp3/tools/*.py` module.
2. Add the JSON schema to `TOOL_SCHEMAS` in `src/wasp3/tools/__init__.py`.
3. Add a `case "tool_name":` branch in `_execute()` in the same file.
4. Add a tier entry (`AUTO` / `ASK` / `ALWAYS_ASK`) in `TOOL_TIERS` in `src/wasp3/permissions.py`.
5. If the tool must be blocked in `--production` mode, add it to `_PRODUCTION_BLOCKED`.

## Switching LLM Providers
Set `WASP3_LLM_PROVIDER` env var. Default is `anthropic`.
Set `WASP3_MODEL` to override the model name.
New providers: implement `LLMClient` protocol in `src/wasp3/llm/` and register in `cli.py`.

## Environment Variables
- `ANTHROPIC_API_KEY` — Required for Anthropic provider
- `WASP3_LLM_PROVIDER` — LLM provider (default: anthropic)
- `WASP3_MODEL` — Model name override (default: claude-sonnet-4-5)

## Testing Wasp3 Itself
```
pytest tests/
```
Tests live in `tests/`. Use `pytest -x` to stop on first failure, `-k <name>` to filter.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arctic-coast)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arctic-coast)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

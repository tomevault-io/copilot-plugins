## pydantic-ai-demos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This repository contains small, focused POCs and demos for learning Pydantic AI. Each example should demonstrate a single concept clearly without unnecessary complexity.

## Project Structure

```
pyd-ai/
├── src/core/          # Shared utilities (model config, common helpers)
├── demos/
│   ├── basics/        # Fundamental concepts
│   ├── tools/         # Tool usage demos
│   ├── streaming/     # Streaming responses
│   └── <category>/    # Add new categories as needed
│       └── <demo>/    # Multi-file demos get their own subfolder
```

## Commands

```bash
# Initial setup (creates venv, installs deps, installs project in editable mode)
uv sync

# Run a demo
uv run python demos/basics/hello.py

# Add dependencies
uv add <package>

# Sync dependencies
uv sync
```

## Guidelines

- **Keep POCs minimal** - Each demo should focus on one idea and drive it home
- **Avoid bloat** - No unnecessary abstractions, helpers, or over-engineering
- **Self-contained examples** - Each script should be runnable independently
- **Use Pydantic AI patterns** - Reference `/pydantic-ai` skill for framework guidance
- **Shared code** - Put reusable utilities in `src/core/`, import as `from core import ...`
- **Multi-file demos** - Create a subfolder within the category

## Pydantic AI Skill Feedback

If you encounter difficulties with the `/pydantic-ai` skill (missing information, outdated patterns, unclear examples), spawn a sub-agent to update the skill files at `~/.claude/skills/pydantic-ai/` with improvements based on what you learned. This helps enhance the skill over time.

## Pydantic AI Skill: Quality Charter

The bespoke `~/.claude/skills/pydantic-ai/` skill is curated from this repo. When improving it, hold to:

- **Bespoke skill is the source of truth.** The official `building-pydantic-ai-agents` skill is a *structural reference only* (navigation/decision-trees) — never a content source; it lags and shows deprecated patterns as current.
- **Verify before encoding.** Check every claim/example against the *actually installed* `pydantic_ai` version; each reference file carries a "Verified against pydantic-ai X.Y.Z" stamp.
- **Maintain deprecation→replacement maps.** Prefer the modern surface: capabilities over constructor kwargs; properties over deprecated callables (e.g. `result.usage`, not `result.usage()`); `openai-chat:`/`openai-responses:` over bare `openai:`.
- **Depth over breadth.** Cover the hard/bleeding-edge surface (harness/capabilities, graphs, streaming) thoroughly rather than skimming everything.
- **Use anti-hallucination "this does NOT exist" callouts deliberately.**
- **On every framework bump, re-verify** stamped files against the new version and update deprecation maps.

Full comparison vs the official skill and the resolved `result.usage` contradiction: see `docs/pydantic-ai-modernization.md` §8.

---
> Source: [Mazyod/pydantic-ai-demos](https://github.com/Mazyod/pydantic-ai-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

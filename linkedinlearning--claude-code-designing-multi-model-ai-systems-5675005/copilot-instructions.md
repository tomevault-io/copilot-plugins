## claude-code-designing-multi-model-ai-systems-5675005

> Multi-model AI help desk assistant for **TaskFlow**, a fictional project management tool (like Asana). This is a LinkedIn Learning course project ("Claude Code: Designing Multi-Model AI Systems"). The architecture evolves with each course module — expect structural changes as new concepts are introduced.

# CLAUDE.md

## Project Overview

Multi-model AI help desk assistant for **TaskFlow**, a fictional project management tool (like Asana). This is a LinkedIn Learning course project ("Claude Code: Designing Multi-Model AI Systems"). The architecture evolves with each course module — expect structural changes as new concepts are introduced.

## Tech Stack

- Python 3 (source code in `src/`)
- Anthropic API (Claude models)
- API key stored in `.env` as `ANTHROPIC_API_KEY`

## Running

```
python src/helpdesk_bot.py --debug
```

## Project Structure

```
src/          # All Python source code
prompts/      # Prompt templates and system prompts
diagrams/     # Architecture and flow diagrams
```

## Key Conventions

- Keep all Python code in `src/`
- Use the Anthropic Python SDK (`anthropic` package)
- The architecture is iterative — each module may restructure or extend prior work
- Virtual environment lives in `venv/` (gitignored)

---
> Source: [LinkedInLearning/claude-code-designing-multi-model-ai-systems-5675005](https://github.com/LinkedInLearning/claude-code-designing-multi-model-ai-systems-5675005) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->

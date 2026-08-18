## ai-engineering

> This is an AI engineering learning repo following the AEGIS curriculum. It is not a production application.

# AGENTS.md — ai-engineering

## Project structure

This is an AI engineering learning repo following the AEGIS curriculum. It is not a production application.

| Path | Purpose |
|---|---|
| `experiments/week[1-4]-*/` | Weekly learning experiments with standalone scripts |
| `projects/p1-cli-chatbot/` | CLI chatbot (in progress, uses PydanticAI) |
| `projects/p2-*`, `p3-*`, `p4-*` | Future projects (not started) |
| `playground/` | Scratch notes |
| `.obsidian/` | Obsidian vault config — do not delete |

## Toolchain

- **Package manager:** `uv` (`uv sync`, `uv add`, `uv lock`) — do not use pip
- **Python:** 3.10 (see `.python-version`)
- **Env vars:** Create `.env` from `.env.example`. Scripts call `load_dotenv()` which searches from CWD upward, so run everything from repo root.
- **No test/lint/typecheck/CI config exists.** Do not look for it or add it unless asked.
- **`*.so` files are build artifacts.** Do not commit them (already in `.gitignore`).

## Running scripts

All experiment scripts are standalone and run from repo root:

```bash
uv run python experiments/week4-pydanticai/scripts/01_basic_agent.py
```

## API key quirks

Two different patterns exist. Use the correct one based on which experiment directory you are in:

- **`week1-api-basics/`, `week4-pydanticai/`, `projects/p1-cli-chatbot/`** — use `OPENAI_API_KEY` with direct OpenAI endpoint
- **`week3-pydantic/`** — uses `FREELLMAPI_API_KEY` with local proxy at `base_url="http://localhost:3001/v1"`

Each experiment directory has its own `config.py`. Check it before running scripts.

## Async pattern

PydanticAI agents use `asyncio.run(main())`. Regular OpenAI SDK scripts do not.

## Project 1 notes

`projects/p1-cli-chatbot/scripts/chatbot.py` currently has mock responses. The `respond()` method still needs to be wired to a PydanticAI agent or OpenAI completion call.

---
> Source: [sean-ngwenya/ai-engineering](https://github.com/sean-ngwenya/ai-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->

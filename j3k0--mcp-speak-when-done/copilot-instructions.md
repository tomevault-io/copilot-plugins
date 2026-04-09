## mcp-speak-when-done

> mcp-speak-when-done is a PyPI-published MCP server that makes AI agents speak a brief summary of every response. It calls an OpenAI-compatible TTS API, caches audio locally, and plays it.

# CLAUDE.md

## Project Overview

mcp-speak-when-done is a PyPI-published MCP server that makes AI agents speak a brief summary of every response. It calls an OpenAI-compatible TTS API, caches audio locally, and plays it.

## Running

```bash
# Direct (stdio transport, default)
OPENAI_API_KEY="key" python mcp_speak_when_done.py

# SSE transport (for remote use)
OPENAI_API_KEY="key" python mcp_speak_when_done.py --transport sse

# Installed via uvx
uvx mcp-speak-when-done
```

There are no build steps, tests, or linters. The module is run directly.

## Architecture

Single file: `mcp_speak_when_done.py`

- **Provider detection** — auto-detects Groq (`gsk_` prefix) vs OpenAI from API key, sets model/voice/URL defaults
- **Config** — env vars override auto-detected defaults
- **Cache** — platform cache dir (`platformdirs`), SHA256-hashed filenames, 24h auto-cleanup
- **TTS** — httpx POST to OpenAI-compatible API with retry/backoff
- **Playback** — ffplay/mplayer/vlc
- **Proxy mode** — when `SPEECH_PROXY_URL` is set, forwards tool calls to a remote MCP server over SSE instead of doing local TTS
- **Entry point** — `main()` uses argparse for `--transport`, creates FastMCP instance, registers tool, runs server

## Key Design Decisions

- Tool always returns "Done" even on error (must not break the agent)
- FastMCP instance created inside `main()` (not module-level) because port is set at construction time
- Tool function named `speak_when_done` so FastMCP auto-registers it with that name (proxy mode forwards by name)
- WAV format forced for all providers (simplifies caching)

## Dependencies

`mcp[cli]`, `httpx`, `platformdirs`

## Packaging

- `pyproject.toml` with `[tool.setuptools] py-modules = ["mcp_speak_when_done"]`
- Entry point: `mcp-speak-when-done = "mcp_speak_when_done:main"`
- Published to PyPI, installed via `uvx mcp-speak-when-done`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j3k0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/j3k0)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

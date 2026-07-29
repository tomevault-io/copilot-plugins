## ogx-demos

> Provides model discovery and document helpers used by most demos:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of hands-on demos for [OGX](https://github.com/ogx-ai/ogx) — an open-source AI platform. Demos cover chat completions, RAG, agents, observability, OpenAI compatibility, and end-to-end applications. All demos run locally in the terminal against a running OGX server.

## Development Setup

```bash
uv sync                                          # install dependencies
ollama pull llama3.2:3b                           # pull a model
uvx --from 'ogx[starter]' ogx run starter        # start OGX server (port 8321)
```

Requires Python >= 3.12 and [uv](https://docs.astral.sh/uv/).

## Running Demos

Demos use `python-fire` for CLI args. Every demo takes `host` and `port` positionally:

```bash
uv run python -m demos.01_foundations.01_client_setup localhost 8321
uv run python -m demos.01_foundations.02_chat_completion localhost 8321 --stream
```

The `06_openai_compatibility` demos use `sys.path` manipulation for imports (not the `demos.` package path), but should still be run the same way.

Model can be specified with `--model_id` or the `OGX_MODEL` env var. If omitted, demos auto-detect the first available chat model.

## Key Libraries

- **`ogx_client`** (`OgxClient`, `Agent`, `AgentEventLogger`) — native OGX SDK for inference, vector IO, tools, agents
- **`openai`** (`OpenAI`) — used in `06_openai_compatibility` demos against OGX's `/v1` endpoint
- **`python-fire`** — CLI argument parsing for all demos
- **`python-dotenv`** — optional `.env` loading (gracefully skipped if missing)

## Architecture

### Demo Organization

Demos are in `demos/` under numbered directories that represent progressive phases:
- `01_foundations` — client setup, chat, vectors, tools, MCP
- `02_responses_basics` — Responses API (tool calling, streaming, structured output)
- `03_rag` — RAG patterns (file search, multi-source, metadata, chunking)
- `04_agents` — agent patterns (chat, multimodal, tools, RAG agents, routing)
- `05_observability` — OpenTelemetry/Jaeger/Prometheus/Grafana setup (shell scripts + configs, not Python)
- `06_openai_compatibility` — using the OpenAI SDK against OGX's `/v1` endpoints

Additional directories outside the numbered progression:
- `demos/client_tools/` — custom tool implementations (calculator, ticker, web search)
- `demos/rag_eval/` — RAG evaluation notebook
- `demos/shared/` — shared utilities used across demos

### Shared Utilities (`demos/shared/utils.py`)

Provides model discovery and document helpers used by most demos:
- `check_model_is_available()`, `get_any_available_chat_model()`, `get_any_available_embedding_model()` — model resolution for `OgxClient`
- `resolve_openai_model()` — model resolution for `OpenAI` client
- `download_documents()`, `build_context()` — document download and RAG context formatting

### Two Client Patterns

1. **OGX native** (most demos): `OgxClient(base_url=f"http://{host}:{port}")`
2. **OpenAI-compatible** (`06_openai_compatibility`): `OpenAI(base_url=f"{scheme}://{host}:{port}/v1", api_key="fake")`

### OGX API Routes

The Agents API routes are at `/responses/*`, not `/agents/*`. The Responses API is accessed via `/v1` for OpenAI compatibility.

## Tests

- `tests/eval_tests/` — tool-calling evaluation harness against MCP servers (ansible, github, openshift, custom). Uses query JSON files in `tests/eval_tests/queries/`.
- `tests/scripts/` — standalone integration scripts for agents with RAG and MCP tools.

No pytest suite or test runner configured — tests are run as standalone scripts.

## CI

GitHub Actions runs pre-commit hooks on PRs to `main` (trailing whitespace, end-of-file fixer, large file check).

## Environment Variables

Copy `.env.example` to `.env`. Key variables:
- `REMOTE_BASE_URL` — OGX server URL for remote deployments
- `TAVILY_SEARCH_API_KEY` — required for agent demos with web search
- `VDB_PROVIDER`, `VDB_EMBEDDING` — vector DB config for RAG demos
- `OGX_MODEL` — override default model selection

## Deployment

`deployment/kubernetes/` contains manifests for Kind cluster deployment: vLLM model servers (with x86_64/arm64 overlays), OGX server CR, and MCP servers (math-mcp example with Dockerfile).

---
> Source: [ogx-ai/ogx-demos](https://github.com/ogx-ai/ogx-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->

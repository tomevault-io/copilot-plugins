## groktocrawl

> Guide for AI coding agents working on the GroktoCrawl codebase.

# AGENTS.md

Guide for AI coding agents working on the GroktoCrawl codebase.

## Project Overview

GroktoCrawl is a self-hosted, MIT-licensed alternative to Firecrawl. It implements the Firecrawl v2 API surface as a set of Python FastAPI services running in Docker.

## Repo Structure

```
groktocrawl/
├── agent-svc/          # Main API + agent research loop
│   └── agent/
│       ├── app.py      # FastAPI app factory, wires dependencies
│       ├── api.py      # Route handlers (all endpoints)
│       ├── models.py   # Pydantic request/response schemas
│       ├── worker.py   # Job processing functions (async)
│       ├── research.py # Agent research loop (search → scrape → LLM)
│       ├── scraper_client.py  # HTTP client to scraper-svc
│       ├── searxng_client.py  # Search API client
│       ├── llm.py      # OpenAI-compatible LLM client
│       └── store.py    # Job CRUD backed by Valkey
├── scraper-svc/        # URL → markdown service
│   └── scraper/
│       ├── app.py      # FastAPI, single /scrape endpoint
│       ├── fetch.py    # Three-tier fetch strategy
│       └── extract.py  # HTML → markdown conversion
├── search-svc/         # Search fixture for local testing
├── llm-svc/            # LLM fixture for local testing
├── test-site/          # Fixture website for integration tests
├── tests/
│   └── test_stack.py   # Integration tests
└── docker-compose.yml  # Single-file deployment
```

## Key Architecture Decisions

### Inline async processing (no RQ worker)

Jobs are processed with `asyncio.create_task()` inside the API process. This avoids needing a separate worker container. For production deployments with high throughput, restore the RQ queue and add a worker container.

### Three-tier smart scraper

Tier 1: Check `/llms.txt` at the site root (one GET, whole site in markdown)
Tier 2: Request with `Accept: text/markdown` header (per-page markdown)
Tier 3: Playwright render + readability extraction

### LLM-agnostic

The agent service uses an OpenAI-compatible client. Swap the provider by changing `LLM_BASE_URL`, `LLM_API_KEY`, and `LLM_MODEL` in `.env`.

**Per-request model override:** `POST /v2/agent` accepts an optional `model` field in the request body. When set to a model name (e.g., `"gpt-4o"`), it overrides `LLM_MODEL` for that job. When omitted or `"default"`, the env-configured model is used. Wired through in `api.py` → `worker.py` → `research.py`.

**System prompt:** The agent's research behavior is defined by `SYSTEM_PROMPT` and `EXTRACT_SYSTEM_PROMPT` constants in `research.py`. These are fixed — not configurable at runtime. They instruct the LLM to evaluate source quality, synthesize across pages, detect contradictions, and cite sources.

## Testing

```bash
# Full integration test against running Docker stack
cp .env.sample .env
docker compose up --build -d
docker compose exec agent-svc python3 /app/agent/tests/test_stack.py

# Or run inline (requires httpx)
pip install httpx
python tests/test_stack.py
```

The integration tests in `tests/test_stack.py` verify all endpoints against a live Docker stack with fixture services.

## Making Changes

1. Edit the relevant service code under `agent-svc/` or `scraper-svc/`
2. Rebuild: `docker compose build <service>`
3. Recreate: `docker compose up -d --force-recreate <service>`
4. Test: run the integration tests

## Adding a New Endpoint

1. Add the route handler in `agent-svc/agent/api.py`
2. Add request/response models in `agent-svc/agent/models.py`
3. If the endpoint is async (returns a job ID), **it must accept a `webhook` field** and fire it on completion/failure via `deliver_webhook()` in `agent/webhook.py`
4. Rebuild the agent-svc image
5. Add a test case in `tests/test_stack.py`

## Environment Variables

See `.env.sample` for all configurable variables. The `.env` file is loaded by `docker compose` automatically via the `env_file:` directive in `docker-compose.yml`.

---
> Source: [groktopus/groktocrawl](https://github.com/groktopus/groktocrawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->

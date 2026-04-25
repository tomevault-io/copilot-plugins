## restful-api-testing-framework

> Global behavior & expectations for Python Agentic Multi-Agent apps (FastAPI + Pydantic + LangGraph/LangChain + ADK).


# General AI Behavior

- Read and respect repository context (README, ADRs, `pyproject.toml`, `./agents/`, `./app/`, `./configs/`).
- Read and respect repository context (README, ADRs, `pyproject.toml`, `./agents/`, `./app/`, `./infra/`, `./domain/`, `./application/`, `./adapters/`).
- Prefer explicit, opinionated guidance; choose one good approach and justify with trade-offs.
- Default to concurrency-safe, async-first designs.
- Keep responses concise; avoid filler; show critical commands/config first, then rationale.
- Prefer progressive enhancement: minimal viable design → resilience → observability → optimizations.

# Architecture & Dependency Rules (summary)

- Hexagonal (ports/adapters):
  - `domain/` contains entities, value objects, domain services, and ports (interfaces).
  - `application/` contains use-cases/orchestrators that depend only on domain ports.
  - `adapters/` implement ports (llm, rest, reporting, cache, logging). No business logic here.
  - `app/` is the HTTP edge (routers, deps, DTOs, middleware) calling application use-cases via DI.
  - `infra/` wires DI, configs, and observability.
- Dependency direction: app/adapters/infra → application → domain; never the reverse.

# Communication & Deliverables

- Respond in English with production-quality code, docstrings, and comments when non-obvious.
- For each non-trivial change, include: rationale, alternatives considered, and rollback note.
- When generating multiple files, include filenames/paths and tree structure.

# Defaults & Versions

- Python 3.12, `venv` for isolation.
- Code style: Ruff (linter/fixes), Black (format), isort (imports), MyPy/Pyright (typing).
- Testing: Pytest + pytest-asyncio; coverage target ≥ 85% critical paths.
- Observability: OpenTelemetry (traces/metrics) + structured logs.
- Security: OAuth2/JWT for APIs, least-privilege tokens for tools, secrets from env/secret store (no plaintext).

# Performance & Reliability First Principles

- Timeouts everywhere; retries only for idempotent ops; use exponential backoff with jitter.
- Bound concurrency (semaphores/bulkheads) for tool calls and IO.
- Cache hot paths (in-memory + Redis) with explicit TTL and invalidation strategy.
- Apply rate limiting and request size limits at API edge.
- Prefer graceful degradation with typed fallbacks and partial results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HZeroxium) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

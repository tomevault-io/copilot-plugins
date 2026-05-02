## projectrule

> - This repo implements a **Warehouse Operational Assistant** aligned to NVIDIA AI Blueprints (multi-agent planner + Inventory, Operations, Safety; RAG over Postgres/Timescale + Milvus; Guardrails; API Gateway).

# Project Context
- This repo implements a **Warehouse Operational Assistant** aligned to NVIDIA AI Blueprints (multi-agent planner + Inventory, Operations, Safety; RAG over Postgres/Timescale + Milvus; Guardrails; API Gateway).
- Primary services: `chain_server/` (FastAPI + LangGraph), `inventory_retriever/` (structured + vector), `memory_retriever/`, `guardrails/`, `ui/`.

# Development Environment
- Use Python virtual environment (`env/`) for all development work
- Follow conventional commit messages: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`
- Run linting (black, flake8, mypy) before commits
- Write tests for all new features (aim for 80%+ coverage)
- Use `.env.example` for required environment variables; never commit `.env` files

# Tech & Tooling
- **Python 3.11+**, **FastAPI**, **Pydantic v2**, **LangGraph**; DB via **psycopg**.
- Infra (dev): Timescale/Postgres (host port **5435**), Redis (6379), Kafka (9092), Milvus (19530). DSNs come from `.env`.
- Use type hints everywhere; write small, testable functions.
- Validate all required env vars on startup; use different configs for dev/staging/prod.

# Architecture & Services
- Prefer composition over inheritance; clear boundaries (`services/llm`, `services/retriever`, `services/memory`).
- Implement health checks for all services (`/health`, `/ready`)
- Use async/await for I/O operations; avoid blocking calls
- Monitor memory usage and implement circuit breakers
- Set up proper metrics collection (Prometheus/Grafana)

# API Design Standards
- All endpoints under `/api/v1/*`. Routers live in `chain_server/routers/`.
- Return Pydantic models; never return raw DB rows.
- Use HTTP status codes correctly (200, 201, 400, 404, 500)
- Implement proper pagination for list endpoints
- Include request/response validation with detailed error messages
- Implement rate limiting and request throttling
- Document API endpoints with OpenAPI/Swagger

# Error Handling & Logging
- Use structured error responses with error codes and context
- Implement proper logging levels (DEBUG, INFO, WARN, ERROR)
- Never expose internal errors to clients; use generic messages
- Include correlation IDs for request tracing
- Log with context, not secrets; prefer structured logs

# Data Management
- **Structured retriever** → Postgres/Timescale SQL (parameterized queries only).
- **Vector retriever** → Milvus; hybrid ranking; pass **Context Synthesis** to LLM.
- Keep schema changes in `data/postgres/` (migrations) and Timescale hypertables for telemetry.
- Use database transactions for multi-step operations
- Implement proper connection pooling
- Version all database migrations with timestamps
- Use read replicas for heavy query workloads

# Security & Compliance
- No secrets in code; read from env; validate all inputs; sanitize prompts
- Implement proper authentication (JWT/OAuth2)
- Use HTTPS in production
- Validate and sanitize all user inputs
- Implement CORS policies
- Use secrets management for sensitive data
- Regular security audits and dependency updates

# Testing & Quality
- Tests go under `tests/` mirroring package layout; write unit + e2e where feasible
- Unit tests: Test individual functions and classes
- Integration tests: Test service interactions
- E2E tests: Test complete user workflows
- Use pytest fixtures for test data setup
- Mock external services in tests
- Run linting (black, flake8, mypy) before commits

# Performance & Monitoring
- Implement health checks for all services (`/health`, `/ready`)
- Use async/await for I/O operations; avoid blocking calls
- Monitor memory usage and implement circuit breakers
- Set up proper metrics collection (Prometheus/Grafana)
- Use appropriate caching strategies
- Implement proper timeout handling

# Documentation
- Use docstrings for all public functions/classes
- Include type hints in all function signatures
- Document API endpoints with OpenAPI/Swagger
- Keep README.md updated with setup instructions
- Update `docs/architecture/diagrams/` with **NVIDIA blueprint style** diagrams when components change
- Maintain ADRs under `docs/architecture/adr/` for significant decisions

# Deployment & Operations
- Use Docker for containerization
- Implement proper CI/CD pipelines
- Use environment-specific configurations
- Monitor application performance and errors
- Implement proper backup and recovery procedures

# What NOT to do
- No ad-hoc SQL string interpolation; no hard-coded ports/credentials; no endpoints outside `/api/v1`
- No blocking I/O operations in async functions
- No secrets or sensitive data in logs or error messages
- No direct database access from API endpoints (use service layer)
- No untested code in production

---
> Source: [NVIDIA-AI-Blueprints/Multi-Agent-Intelligent-Warehouse](https://github.com/NVIDIA-AI-Blueprints/Multi-Agent-Intelligent-Warehouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->

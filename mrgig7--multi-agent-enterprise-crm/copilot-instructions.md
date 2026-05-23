## multi-agent-enterprise-crm

> - `frontend/` — Next.js 14 UI with Tailwind; auth-aware dashboards and agent consoles.

# Repository Guidelines

## Project Structure & Module Organization
- `frontend/` — Next.js 14 UI with Tailwind; auth-aware dashboards and agent consoles.  
- `gateway/` — Express + TypeScript API gateway with Prisma schemas in `prisma/` and Jest tests in `tests/`.  
- `agents/` — LangGraph/Ollama Python agents; entrypoint `python -m src.orchestrator.main`; tests live in `agents/tests/`.  
- `core_services/` — Shared Python services (cache, DR, governance, event store).  
- `database/migrations/` — SQL migrations incl. RLS/outbox.  
- `policies/` — OPA/Rego policies for RBAC/ABAC.  
- `observability/` — Prometheus config and Grafana dashboards.  
- `deploy/`, `scripts/`, `docs/`, `tests/` — deployment manifests, helper scripts, documentation, and integration/chaos suites.  
- `docker-compose.yml` — brings up the full stack for local dev.

## Build, Test, and Development Commands
```bash
# Bring up core infra
docker-compose up -d postgres redis kafka opa

# Frontend
cd frontend && npm install && npm run dev        # hot-reload UI
npm run build && npm run start                   # production build
npm run lint                                     # Next.js lint

# Gateway
cd gateway && npm install
npm run dev                                      # ts-node + nodemon
npm run build                                    # prisma generate + tsc
npm test                                         # Jest unit/integration
npm run lint                                     # ESLint (TS rules)

# Agents (Python)
cd agents && pip install -r requirements.txt
python -m src.orchestrator.main                  # start LangGraph agents
pytest agents/tests -v                           # agent tests

# End-to-end / governance
pytest tests/test_gdpr_forget.py tests/test_data_export.py -v
```

## Coding Style & Naming Conventions
- **TypeScript/JavaScript:** ESLint configs live in `gateway/.eslintrc.js` and `frontend/.eslintrc.json` (Next core web vitals). Prefer TypeScript, 2-space indent, single quotes, and explicit exports. Avoid unused vars; underscore unused args.  
- **Python:** Follow PEP 8, 4-space indent. Favor type hints and structured logging (`structlog`).  
- **Files/dirs:** kebab-case for folders (`event-log`), PascalCase for React components, camelCase for vars/functions, SCREAMING_SNAKE_CASE for env vars.  
- Run `npm run lint` (frontend/gateway) before commits; add formatting if your editor supports it (Prettier optional but keep lint clean).

## Testing Guidelines
- Target: keep existing pytest/Jest suites green; add regression tests with new features.  
- Name tests by behavior: `test_<module>_<condition>_<expected>()` for pytest; `<UnitOfWork>_returns...` for Jest.  
- Use integration tests in `tests/` for GDPR/export and chaos; set `CHAOS_TESTS_ENABLED=true` only when running locally.  
- For gateway DB changes, run `prisma migrate dev` and regenerate clients before tests.

## Commit & Pull Request Guidelines
- Commits: short imperative titles (<72 chars), e.g., `Fix gateway Docker build`. Group related changes.  
- PRs: include what/why, linked issue or Jira ID, test evidence (`npm test`, `pytest …`), and screenshots/GIFs for UI changes. Note schema or env changes prominently. Keep PRs scoped and rebased on main.

## Security & Configuration Tips
- Never commit secrets; use `.env` (copy from `.env.example`).  
- Enforce tenant safety: RLS migrations must run after `docker-compose up`.  
- OPA policies under `policies/` gate sensitive actions—update docs when changing them.  
- Telemetry is enabled (OpenTelemetry, Prometheus); keep spans/metrics consistent when adding services.

---
> Source: [Mrgig7/Multi-Agent-Enterprise-CRM](https://github.com/Mrgig7/Multi-Agent-Enterprise-CRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->

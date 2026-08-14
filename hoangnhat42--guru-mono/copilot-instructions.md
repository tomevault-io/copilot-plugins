## guru-mono

> - `docker-compose.yml` wires PostgreSQL, React/Nginx, Go, and Python.

# Agent Guide

## Start at the wiring

- `docker-compose.yml` wires PostgreSQL, React/Nginx, Go, and Python.
- `backend-go/main.go` owns API composition, migrations, process recovery, and
  the preview proxy.
- `ai-python/src/main.py` owns AI routes and pipeline lifecycle.
- `frontend-react/src/main.tsx` and `frontend-react/src/App.tsx` own the web
  entry point and routes.

Trace behavior through imports and API/SSE contracts before relying on planning
documents. For cross-service work, verify every caller and consumer.

## Checks

```bash
(cd backend-go && go test ./...)
(cd frontend-react && bun install --frozen-lockfile && bun run build)
(cd ai-python && python -m pytest)
docker compose -p guru-ai config --quiet
```

Use `docker compose -p guru-ai`; sandbox network and volume names depend on that
project name.

## Guardrails

- Never commit secrets or populated environment files.
- Add a new database migration instead of editing an applied migration.
- Keep generated React and Go templates aligned with runtime contracts.
- Preserve SSE event names and authenticated route behavior unless a migration
  plan is part of the change.
- Keep changes focused and cite file-level evidence in handoffs.

---
> Source: [hoangnhat42/guru-mono](https://github.com/hoangnhat42/guru-mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

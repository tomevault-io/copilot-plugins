## p7erp-local

> - NEVER hardcode `localhost` or `127.0.0.1` into the `docker-compose.yml` environment blocks. Always use dynamic variables (e.g., `${VITE_API_BASE_URL:-http://localhost:8000}`) so the configuration can adapt to production environments.

# Prime7 ERP Production Deployment Rules

- NEVER hardcode `localhost` or `127.0.0.1` into the `docker-compose.yml` environment blocks. Always use dynamic variables (e.g., `${VITE_API_BASE_URL:-http://localhost:8000}`) so the configuration can adapt to production environments.
- The React frontend relies on Vite's HMR. When modifying `vite.config.ts`, always ensure `server.hmr.clientPort` gracefully falls back to `5173` for local development, while allowing `443` for secure production reverse proxies (Nginx).
- CORS headers in the FastAPI backend must always remain configurable via the `CORS_ORIGINS` environment variable. Do not hardcode allowed origins in `main.py`.
- Production database URLs must be read from `DATABASE_URL` via the `.env` file, leveraging Docker's internal DNS (e.g., `@postgres:5432`) rather than external IPs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prime7biz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

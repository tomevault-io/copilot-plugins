## start-app-docker-check

> When starting the Wasp app or dev server, verify Docker and Postgres are up first.


# Start app / Wasp dev server

When the user asks to **start the app**, **start Wasp**, **run the dev server**, or similar:

1. **Check Docker** — Run `docker ps` (or `docker info`). If the daemon is unreachable, tell them to start **Docker Desktop** and wait until it is ready. On macOS you may run `open -a Docker`, then re-check after a short wait.
2. **Start Postgres** — From the project root, if the stack is not up: `docker compose up -d` (see `docker-compose.yml`; DB listens on **5433**).
3. **Env** — Ensure `.env.server` exists and `DATABASE_URL` matches the compose credentials (see `.env.server.example`).
4. **Wasp CLI** — Use the repo-pinned Node (see `.node-version`). From automated shells, run Wasp via **`npm run wasp -- start`** or **`bash scripts/with-project-node.sh wasp start`** so `PATH` picks up Node 22 (global `@wasp.sh/wasp-cli`).

Only after Docker responds and the DB container is running should you start Wasp (or tell the user it is safe to do so).

---
> Source: [cvetelinandreev/reShkolo](https://github.com/cvetelinandreev/reShkolo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

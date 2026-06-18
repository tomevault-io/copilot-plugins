## paperclipai-docker

> Paperclip — Self-hosted deployment via Docker with PostgreSQL and Cloudflare Tunnel.

# CLAUDE.md

## Project

Paperclip — Self-hosted deployment via Docker with PostgreSQL and Cloudflare Tunnel.
Source: https://github.com/MadeByAdem/paperclipai-docker

## Build & Run

```bash
cp .env.example .env
# Fill in .env with your values
docker compose up --build -d
```

## Architecture

- `Dockerfile` — Multi-stage build: clones repo, installs deps, builds, creates production image
- `docker-compose.yaml` — 2 services: PostgreSQL 17, Paperclip server (localhost:3100)
- `docker-entrypoint.sh` — UID/GID matching for volume permissions
- Server binds to 127.0.0.1:3100 only — a reverse proxy (e.g. cloudflared) on the host routes external traffic

---
> Source: [Made-By-Adem/paperclipai-docker](https://github.com/Made-By-Adem/paperclipai-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

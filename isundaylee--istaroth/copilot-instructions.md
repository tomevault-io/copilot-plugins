## docker-dev

> Docker Compose dev stack for per-worktree isolation


# Docker dev stack

Never run bare `docker compose up` from `docker-compose/web/`.

- **Start**: `scripts/dev-compose.sh up` (runs detached)
- **URLs**: run `scripts/dev-compose.sh urls` before testing the web UI or API; use the printed `CONDUCTOR_PORT` URL
- **Stop**: `scripts/dev-compose.sh down` (also runs automatically via `sessionEnd` hook when the agent session ends)
- **Setup only** (symlink env files): `scripts/dev-compose.sh setup`

`uv-cache` and `hf-cache` Docker volumes are shared across worktrees intentionally.

---
> Source: [isundaylee/istaroth](https://github.com/isundaylee/istaroth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->

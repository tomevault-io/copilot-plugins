## docker-compose-haproxy

> Use the correct compose file pair for dev vs prod; HAProxy is the browser entrypoint in Docker


# Docker Compose & HAProxy

- **Local dev** commands (`up`, `ps`, `logs`, `exec`, `restart`) should use **both** base + dev overlay:  
  `docker compose -f docker-compose.yml -f docker-compose.dev.yml …`
- **Production / amvara9** uses **`docker-compose.prod.yml`** instead of **`docker-compose.dev.yml`** (same base file). Do not mix overlays.
- **Browser / smoke tests:** Traffic goes through **HAProxy**; use the **published host port** from `docker compose ps` (dev default **4202** unless overridden). **`BASE_URL`** for Puppeteer is typically `http://127.0.0.1:4202`.
- After compose or proxy changes, check **haproxy**, **front**, and **back** logs and run a minimal smoke (**`docs/testing.md`**, **`AGENTS.md`**).

---
> Source: [satisfecho/pos](https://github.com/satisfecho/pos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->

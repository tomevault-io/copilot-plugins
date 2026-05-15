## dnsshooter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

dnsshooter is a small FastAPI web app that turns AdGuard Home DNS query activity into a real-time space-shooter visualization. It is a fork of `ph-intercept` (originally Pi-hole–based) re-pointed at AdGuard Home's control API.

## Run / build

This project is fully self-contained: it builds from `python:3.14-slim` and bundles the frontend assets (vendored from upstream `ph-intercept`) under `static/`. The Python adapter (`app.py`, `core/`) and templates (`templates/`) are this repo's own code. There is no local Python entry point — everything runs inside the Docker container.

```bash
# Build + run (reads ADGUARD_PASSWORD from .env next to compose.yaml)
docker compose up --build -d

# Tail logs
docker compose logs -f dnsshooter

# After editing Python/templates, rebuild
docker compose up -d --build
```

The app listens on port 4653 and expects AdGuard Home reachable at `ADGUARD_URL` (defaults to `http://host.docker.internal:3000/control`, requiring the `host.docker.internal:host-gateway` extra_host that compose.yaml already wires up).

There is no test suite, lint config, or package manifest in this repo.

## Architecture

The Python side is intentionally thin — it polls AdGuard, normalizes results into Pi-hole-shaped payloads, and streams them to the browser, where the heavy starfield/game rendering lives (in the static assets inherited from the ph-intercept base image).

### Request flow

1. `app.py` — FastAPI app with a `lifespan` that owns a single shared `httpx.AsyncClient` (HTTP Basic Auth) and a background `query_poller` task. All AdGuard calls in `core/adguard.py` take that client as a parameter; they don't construct their own.
2. `GET /` renders `templates/index.html` and injects `bg_config` + the AdGuard dashboard URL.
3. `GET /api/pihole/stats` → `get_pihole_stats()` fans out to `/stats`, `/status`, `/filtering/status` in parallel and returns a Pi-hole-compatible shape (`queries`, `blocked`, `percent`, `gravity`, `blocking`, `block_timer`).
4. `POST /api/pihole/toggle` → `toggle_blocking()` POSTs `/protection`. Note: the incoming `timer` is in **seconds** (Pi-hole convention) and is converted to **milliseconds** for AdGuard's `duration` field.
5. `POST /api/pihole/gravity-update` → `trigger_gravity_update()` POSTs `/filtering/refresh` (10s timeout — slower than other calls).
6. `GET /api/pihole/events` is a Server-Sent Events endpoint. Each connected client gets its own `asyncio.Queue` registered via `add_ws_client`. The names `add_ws_client` / `remove_ws_client` are legacy; transport is SSE, not WebSocket.

### Polling & broadcast loop (`core/adguard.py`)

- `query_poller` runs every 0.5s but only fetches when at least one SSE client is connected.
- It pulls `/querylog?limit=50` (newest-first) and uses `_last_q_time` as a watermark so only queries newer than the last broadcast are emitted. On the first poll after `reset_watermark()` (called when a new SSE client connects), it primes the watermark and emits nothing — the first batch arrives on the second tick.
- AdGuard `reason` codes in `_BLOCKED_REASONS` map to `status: "blocked"`; everything else is `"allowed"`. The `source` field is `"blocked" | "cache" | "upstream"` based on `reason` and `cached`.
- `_broadcast` uses non-blocking `put_nowait` per client. On `QueueFull` it pushes a `None` sentinel (the SSE generator treats `None` as "drop this client") and removes the client — backpressure is handled by disconnect, not buffering.

### Translation layer

The frontend (`static/`, vendored from `ph-intercept`) was written for Pi-hole's API; the Python in this repo is the adapter that lets it talk to AdGuard unchanged. Keep response shapes matching Pi-hole's expectations even when AdGuard's native fields differ (e.g. `block_timer: None`, `gravity` summed from filter `rules_count`, percent computed locally). When pulling in updates from upstream `ph-intercept`, only the `static/` tree should change — `app.py`, `core/`, and `templates/index.html` are this repo's own.

### Configuration (`core/config.py`)

All config is env-var driven (see `compose.yaml` for the canonical list). `BG_IMAGE` overrides `BG_MODE` when set. `ADGUARD_DASHBOARD` is derived by stripping `/control` off `ADGUARD_BASE` and is passed to the template as a click target.

---
> Source: [matthijsbro/dnsshooter](https://github.com/matthijsbro/dnsshooter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->

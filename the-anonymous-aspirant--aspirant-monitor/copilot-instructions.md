## aspirant-monitor

> System metrics sidecar for the Aspirant platform. Exposes container stats, disk usage, and volume information via REST API by reading from the Docker socket.

# aspirant-monitor

System metrics sidecar for the Aspirant platform. Exposes container stats, disk usage, and volume information via REST API by reading from the Docker socket.

## Standards

All Aspirant platform repositories follow the conventions in [aspirant-meta](https://github.com/the-anonymous-aspirant/aspirant-meta).

## Quick Reference

- **Language:** Python 3.11 / FastAPI
- **Port:** 8000 (internal), 8085 (external via Compose)
- **Dependencies:** `requirements.txt`
- **Tests:** `pytest tests/ -v`
- **Docker:** `docker build -t aspirant-monitor .`

## Architecture

See `docs/ARCHITECTURE.md` for data flow and design rationale.

## Key Files

| File | Purpose |
|------|---------|
| `app/main.py` | FastAPI app with lifespan |
| `app/routes.py` | Three endpoints: `/health`, `/containers`, `/disk` |
| `app/config.py` | Environment configuration |
| `Dockerfile` | Container build |
| `tests/` | Unit tests with mocked Docker client |

---
> Source: [the-anonymous-aspirant/aspirant-monitor](https://github.com/the-anonymous-aspirant/aspirant-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

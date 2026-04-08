## health-iot-monitor

> Build a simulated ESP32 wearable that sends vitals to a FastAPI backend with PostgreSQL storage, anomaly alerts, and basic dashboards.

# Health IoT Monitoring System (Sim-first, IoT-compatible)

## Goal
Build a simulated ESP32 wearable that sends vitals to a FastAPI backend with PostgreSQL storage, anomaly alerts, and basic dashboards.

## Non-goals (v1)
- No real hardware required (Wokwi only)
- No PHI/real patient data
- No complex ML in v1

## Tech choices
- Backend: FastAPI + SQLAlchemy + Alembic
- DB: PostgreSQL
- Local dev: Docker Compose
- Simulator: Wokwi ESP32 (HTTP)
- Optional later: Redis/Celery, simple dashboard

## Core features (v1)
1) Device ingestion endpoint with API key auth (X-DEVICE-KEY)
2) Store readings (heart_rate, temperature, optional spo2) as time-series
3) Alert engine: fever/tachycardia/(optional low spo2) + offline device
4) Dashboard APIs: latest, history, alerts, device status

## Engineering rules
- Always propose a small plan before big edits.
- Keep endpoints RESTful and typed (Pydantic models).
- Add migrations for schema changes.
- Add indexes for (device_id, recorded_at).
- Include a seed script to register a device + generate API key.
- Every major feature includes minimal tests.

## Common commands
- docker compose up --build
- pytest
- alembic upgrade head

## Repo layout
/backend
/simulator
/docker-compose.yml
/README.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nebullii)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nebullii)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

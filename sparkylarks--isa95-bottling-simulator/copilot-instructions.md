## isa95-bottling-simulator

> ISA-95 compliant bottling line demo for Amarach StackWorks / Crosshaven plant.

# Amarach Demo — Cursor Rules

## Project Context
ISA-95 compliant bottling line demo for Amarach StackWorks / Crosshaven plant.
Two Python FastAPI services: erp-shim (port 8765) and oee-service (port 8766).
PostgreSQL database. Node-RED orchestration via MQTT.

## Naming Conventions — STRICT
- Timestamps: ISO8601 UTC string, column/field suffix _ts_utc
- IDs: snake_case, suffix _id (line_id, lot_id, order_id)
- Durations: always minutes, suffix _min (running_min, stopped_min)
- Booleans: prefix is_ (is_complete, is_degraded)
- Status codes: ALL_CAPS enum (PLANNED, RELEASED, IN_PRODUCTION, COMPLETED)
- Line references: always line_id, never just 'line'
- Data source flag: always data_source ('simulator', 'aerogen', 'tulip')

## Code Style
- Python 3.11+
- FastAPI with Pydantic v2
- asyncpg for PostgreSQL (not SQLAlchemy)
- All endpoints return consistent envelope: {status, data, error}
- No print() — use Python logging

## Never Do
- Never use datetime objects in API responses — always ISO string
- Never hardcode connection strings — always from config.py / environment
- Never store OEE as a derived number without storing the components

---
> Source: [SparkyLarks/ISA95-bottling-simulator](https://github.com/SparkyLarks/ISA95-bottling-simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->

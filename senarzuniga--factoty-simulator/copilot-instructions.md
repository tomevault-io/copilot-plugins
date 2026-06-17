## factoty-simulator

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project at a glance

- Domain: digital factory simulator for a corrugated cardboard plant.
- Main package: `dcfs/`.
- Public surfaces: CLI simulation runner, FastAPI server, and Streamlit dashboard.

## First commands to run

1. Install dependencies:
   - `python -m pip install -r requirements.txt`
2. Run tests (unittest-based):
   - `python -m unittest discover -s tests -p "test_*.py"`
3. Optional run commands:
   - CLI simulator: `python -m dcfs.main --mode real_time`
   - API server: `python -m uvicorn dcfs.api.server:app --host 0.0.0.0 --port 9000`
   - Streamlit UI: `python -m streamlit run streamlit_app.py`

## Architecture boundaries

- Simulation core: `dcfs/engine/`
  - `simulator.py` orchestrates step loop and emits normalized events.
  - `factory_state.py` mutates machine state, production, inventory, energy.
  - `event_bus.py` provides async pub/sub event dispatch.
  - `time_engine.py` controls mode and step timing.
- Domain logic: `dcfs/logic/`
  - `failures.py`, `kpis.py`, `requests.py` should stay decoupled and stateless where possible.
- Integrations: `dcfs/integration/`
  - `dep_bridge.py` is attached via simulator step callbacks.
  - `company_profile.py` loads profile config.
- API layer: `dcfs/api/server.py`
  - Wraps a shared `SimulationRuntime` and exposes REST + WS endpoints.
- UI layer: `streamlit_app.py`
  - Runs simulation in background thread with snapshot reads for rendering.

## Code conventions for edits

- Keep event contracts stable unless explicitly requested.
  - Normalized event names are uppercase (`MACHINE_ALERT`, `PRODUCTION_UPDATE`, etc.).
  - Event payloads include IDs and ISO timestamps.
- Prefer small, local changes in the relevant subsystem.
  - Avoid coupling UI/API logic directly into simulation internals.
- Preserve async behavior in engine/API paths.
  - Do not block event loops with long synchronous work.
- Tests use `unittest` (not pytest fixtures by default).
  - Add or update tests in `tests/` for behavior changes.

## Known pitfalls

- `dcfs/api/server.py` currently contains unresolved merge-conflict markers. If your task touches API runtime behavior, inspect and resolve this file first before broader changes.
- README examples include shell-style environment variable usage; adapt commands for the active shell when running locally.

## Key references

- Project overview and endpoint expectations: [README.md](README.md)
- Runtime dependencies: [requirements.txt](requirements.txt)
- Data model seed/schema: [database/schema.sql](database/schema.sql), [database/seed.sql](database/seed.sql)
- Simulation entrypoint: [dcfs/main.py](dcfs/main.py)
- API surface: [dcfs/api/server.py](dcfs/api/server.py)
- Dashboard behavior: [streamlit_app.py](streamlit_app.py)

---
> Source: [senarzuniga/Factoty-Simulator](https://github.com/senarzuniga/Factoty-Simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

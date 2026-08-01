## home-connect-hass

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Connect Alt is a custom Home Assistant integration for BSH Home Connect appliances (Bosch, Siemens, Neff, Constructa). It uses cloud push (SSE) for real-time updates.

**Two-repo architecture:**
- `/workspaces/home-connect-hass` — HA integration (this repo), domain: `home_connect_alt`
- `/workspaces/home-connect-async` — Async Python SDK published to PyPI as `home-connect-async`, used as a dependency

The integration depends on `home-connect-async` (pinned version in `manifest.json`). The SDK handles OAuth2, REST API, SSE streaming, and the appliance data model. The integration maps SDK objects to HA entities.

## Build & Development Commands

```bash
# Format check (CI uses black)
black --check custom_components/

# Format code
black custom_components/

# Run tests (CI config)
pytest tests/ --timeout=9 -n auto

# Validate HA manifest
python -m script.hassfest --integration-path custom_components/home_connect_alt

# Sync translations (after editing en.json)
python tools/sync-translations.py
```

No `requirements_test.txt` currently exists. CI workflow at `.github/workflows/push.yml` defines the test/lint pipeline.

## Architecture

### Integration (`custom_components/home_connect_alt/`)

- **`__init__.py`** — Entry setup, service registration, event coordination. Registers 7 platforms and multiple services (select/start/stop/pause/resume program, set options, apply settings).
- **`common.py`** — `EntityBase`/`InteractiveEntityBase` base classes, `Configuration` manager, `EntityManager`, utility functions shared across platforms.
- **`config_flow.py`** — OAuth2 config flow with API host selection (default, China, simulator).
- **`const.py`** — Domain, API hosts, default settings, device icon mapping, event patterns.
- **`services.py`** — Service handlers for program control automations.

### Entity Platforms

Entities are dynamically created based on appliance data types:
- **`sensor.py`** — Non-boolean status/program/setting values
- **`binary_sensor.py`** — Boolean states (door open, remote control active)
- **`select.py`** — Program selection, enum options, delayed operations
- **`number.py`** — Numeric settings with min/max/step
- **`button.py`** — Actions (start program, debug info)
- **`switch.py`** — Binary toggles (power, child lock)
- **`time.py`** — Delayed operation time picker (absolute time mode)

### SDK (`/workspaces/home-connect-async/home_connect_async/`)

- **`homeconnect.py`** — Main `HomeConnect` coordinator, SSE event loop, appliance discovery
- **`appliance.py`** — Largest module (~45KB). Full appliance data model with programs, options, settings, status, commands
- **`auth.py`** — `AbstractAuth` base (for HA OAuth2 compatibility) and `AuthManager` (standalone OAuth2)
- **`api.py`** — HTTP request dispatcher with retry logic (3 attempts), conditional logging
- **`common.py`** — `ConditionalLogger`, `HomeConnectError`, `HealthStatus` flags

### Data Flow

1. HA config entry → `Configuration` object → platform setup
2. Entities subscribe to appliance callbacks via SDK
3. BSH Cloud API → SSE events → SDK appliance updates → HA entity state changes

## Conventions

- All I/O methods use `async_*` prefix (HA convention)
- Entity names generated from configurable templates (`$brand $appliance - $name`)
- Per-entity and per-appliance configuration overrides supported
- Translation system: 11 languages, `en.json` is the master; run `tools/sync-translations.py` after editing

## Translation Files

Located in `custom_components/home_connect_alt/translations/` (~85KB each). The sync tool copies sensor translations to select entities and maintains key consistency across languages. Always edit `en.json` first, then run the sync script.

---
> Source: [ekutner/home-connect-hass](https://github.com/ekutner/home-connect-hass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->

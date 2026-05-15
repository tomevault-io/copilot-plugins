## homeassistant-evtracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration for EV Tracker — a cloud service that tracks and analyzes electric vehicle charging costs. Supports multiple cars per installation, dual-tariff electricity pricing (time-of-use schedules or entity-based), and session logging with auto-detection of tariff rates and prices.

- **Domain**: `evtracker`
- **Runtime dependency**: `aioevtracker==0.1.0` (async API client library)
- **Python**: 3.12+, Home Assistant 2024.1+
- **IoT class**: Cloud polling (5-minute default interval)

## Commands

### Testing
```bash
pytest -v                                              # all tests
pytest tests/test_services.py -v                       # single file
pytest tests/test_services.py::TestLogSessionService -v  # single class
pytest tests/test_services.py::TestLogSessionService::test_log_session_with_auto_rate_type -v  # single test
pytest --cov=custom_components/evtracker --cov-report=html  # with coverage
```

### Linting & Formatting
```bash
ruff check custom_components/evtracker tests           # lint
ruff check --fix custom_components/evtracker tests     # lint + auto-fix
ruff format custom_components/evtracker tests           # format
```

### Pre-commit
```bash
pre-commit install       # one-time setup
pre-commit run --all-files
```

### Install test dependencies
```bash
pip install -r requirements_test.txt
```

## Architecture

```
Config Flow (config_flow.py)
    → validates API key, lets user select car
    → supports OptionsFlow for tariff schedules, prices, update interval

EVTrackerAPI (api.py)
    → thin wrapper re-exporting aioevtracker.EVTrackerClient as EVTrackerAPI
    → re-exports: ApiError, AuthenticationError, ConnectionError, RateLimitError

EVTrackerDataUpdateCoordinator (coordinator.py)
    → polls API every N seconds (configurable, default 300s)
    → exposes: last_session, current_month, current_year, cars, is_connected

Entities:
├── sensor.py → 8 sensors (monthly/yearly energy+cost, sessions, avg cost, last session energy+cost)
├── binary_sensor.py → connection status + low_tariff (schedule or entity mode)
└── services.py → log_session, log_session_simple (auto-detects rate_type and prices)
```

### Key design decisions

- **Multi-car support**: Each car is a separate ConfigEntry. Services are registered once (on first entry setup) and unregistered when the last entry is removed.
- **Tariff detection**: Three modes — `none`, `schedule` (up to 4 time windows with overnight support, configurable as LOW or HIGH windows, optional weekend-is-always-low), `entity` (mirrors a binary sensor/input_boolean).
- **Price auto-fill**: Services can auto-detect `rate_type` from the low_tariff binary sensor and auto-fill `price_per_kwh` from configured HIGH/LOW prices.
- **Entity unique IDs**: `{car_id}_{sensor_key}` (e.g., `123_monthly_energy`).
- **Config vs Options**: Immutable data (API key, car ID, car name) in `entry.data`; mutable settings (interval, tariff, prices) in `entry.options`.

## Code Conventions

- Fully async codebase — no blocking calls. Uses `aiohttp` via `async_get_clientsession(hass)`.
- All entities extend `CoordinatorEntity` for automatic state updates.
- Line length: 100 chars. Ruff rules: E, W, F, I, B, C4, UP, ARG, SIM, PTH, ERA, RUF.
- Conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `chore:`.
- Version is stored in both `manifest.json` and `const.py` (updated by release CI).
- Tests use `pytest-homeassistant-custom-component` with `AsyncMock` for all API methods. Fixtures are in `tests/conftest.py`.
- `asyncio_mode = "auto"` in pytest config — no need for `@pytest.mark.asyncio` decorators.

---
> Source: [petrmac/homeassistant-evtracker](https://github.com/petrmac/homeassistant-evtracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->

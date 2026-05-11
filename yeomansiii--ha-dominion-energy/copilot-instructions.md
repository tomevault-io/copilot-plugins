## ha-dominion-energy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for monitoring Dominion Energy electricity usage. It provides 30-minute interval data, daily/monthly totals, and cost estimation with multiple calculation modes.

## Architecture

### Key Components

- **`coordinator.py`**: `DominionEnergyCoordinator` extends Home Assistant's `DataUpdateCoordinator`. Manages API calls via the `dompower` library, handles token refresh callbacks, and calculates costs based on configured mode (API estimate, fixed rate, or time-of-use).

- **`config_flow.py`**: Two-step setup flow:
  1. User provides access/refresh tokens (obtained via `dompower auth-helper`)
  2. Integration auto-discovers accounts/meters from API; presents selection UI if multiple exist

  Also implements `OptionsFlow` for cost calculation configuration and reauth flow for expired tokens.

- **`sensor.py`**: Defines sensor entities using `DominionEnergySensorDescription` dataclass with `value_fn` lambdas for extracting data. Sensors include usage metrics, costs, and bill forecast data.

- **`const.py`**: Configuration keys, cost mode constants, and default values.

### Data Flow

1. Config flow validates tokens via `DompowerClient.async_refresh_tokens()`
2. Coordinator fetches interval usage data every 30 minutes
3. Cost calculation uses one of three modes from options: `api_estimate` (derived from bill), `fixed`, or `time_of_use`
4. Sensors read from `DominionEnergyData` dataclass returned by coordinator

### External Dependencies

- **`dompower`**: Python library for Dominion Energy API (required version in `manifest.json`)
- Uses SAP Customer Data Cloud (Gigya) for authentication

## Development Notes

### Home Assistant Integration Patterns Used

- `ConfigEntry.runtime_data` for storing coordinator instance
- Type alias: `type DominionEnergyConfigEntry = ConfigEntry[DominionEnergyCoordinator]`
- Token persistence via `hass.config_entries.async_update_entry()` in callback
- `ConfigEntryAuthFailed` exception triggers reauth flow

### Testing

No test suite currently exists. When adding tests:
- Mock `DompowerClient` responses
- Test cost calculation logic in coordinator
- Test config flow state machine (user -> discover_accounts -> select_meter)

### HACS Distribution

- `hacs.json` configures HACS metadata
- `manifest.json` must be kept in sync with actual `dompower` version requirements

---
> Source: [YeomansIII/ha-dominion-energy](https://github.com/YeomansIII/ha-dominion-energy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->

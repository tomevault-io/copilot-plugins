## gardenasmarthome

> A HACS-compatible Home Assistant custom component for Gardena Smart System API v2. No external Python dependencies — only `aiohttp` (built into HA).

# Gardena Smart System - Home Assistant Integration

## Project Goal
A HACS-compatible Home Assistant custom component for Gardena Smart System API v2. No external Python dependencies — only `aiohttp` (built into HA).

## Architecture
- `custom_components/gardena_smart_system/` — the integration
  - `api/auth.py` — OAuth2 authentication against Husqvarna Group API
  - `api/client.py` — REST API client for locations/devices
  - `api/websocket.py` — WebSocket client for real-time push updates
  - `coordinator.py` — DataUpdateCoordinator, manages devices, WS connections, state
  - `config_flow.py` — UI-based config flow (client_id + client_secret)
  - `lawn_mower.py`, `valve.py`, `sensor.py`, `binary_sensor.py` — HA entity platforms
  - `entities/` — base entity classes
  - `const.py` — constants and service type strings
- `tests/` — pytest tests (conftest.py, test_coordinator.py, test_auth.py, test_config_flow.py)

## Key Design Decisions
- **DEVICE objects from the real API have NO attributes.** Device name, modelType, and serial come from the COMMON service. The coordinator's `_process_included_data()` handles this.
- WebSocket per location with auto-reconnect (exponential backoff)
- Token refresh 5 minutes before expiry
- SSL context created in executor thread (non-blocking)
- **Token reuse:** config flow stores token + expiry in config entry data. Coordinator restores it via `auth.restore_token()` to avoid Husqvarna's simultaneous login rejection.
- **UnitOfIlluminance** not available in all HA versions — use string `"lx"` directly.

## What's Done
- Full integration: config flow, API layer (auth/client/websocket), coordinator, all entity platforms
- API smoke test script (`tests/smoke_test_api.py`)
- Fix: device attributes populated from COMMON service (not DEVICE)
- Fix: simultaneous login error — token passed from config flow to coordinator
- Fix: `UnitOfIlluminance` replaced with `"lx"` string for HA compatibility
- CI: hassfest + pytest + hacs/action in `.github/workflows/validate.yaml`
- README, translations (en/sv), hacs.json, manifest.json
- MIT LICENSE in repo root
- Published to HACS (exists in store as `CorSeptem/GardenaSmartHome`)
- Latest release: **v1.0.3** (manifest.json version matches)
- Default branch: `main`

## What Remains
- Verify all entity platforms work with real devices
- Monitor for further HA compatibility issues

## Development
```bash
pip install -r requirements_dev.txt
pytest tests/ -v
```

## Husqvarna Developer Portal
- Create app at developer.husqvarnagroup.net
- Connect **Gardena Smart System API** under "Connected APIs"
- **Application Key** = Client ID, **Application Secret** = Client Secret
- Redirect URL: `http://localhost` (not used, integration uses Client Credentials flow)

---
> Source: [CorSeptem/GardenaSmartHome](https://github.com/CorSeptem/GardenaSmartHome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->

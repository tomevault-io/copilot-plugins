## kvent-ha

> KVent (Komfovent C4) HA integration — Cursor AI coding rules


# KVent Project Rules

## Architecture
- All Modbus I/O lives in `custom_components/kvent/modbus.py` — **zero HA imports**, independently testable.
- All polling lives in `custom_components/kvent/coordinator.py` via `DataUpdateCoordinator`.
- Entities **never** poll directly; they read from `self.coordinator.data`.
- All entities inherit `CoordinatorEntity`.
- Config entries only — no YAML setup.
- All I/O is `async`. No blocking calls in the event loop.

## const.py
- Zero logic, zero imports.
- All register addresses, mode maps, preset lists, and option dicts go here.

## coordinator.py
- Raises `UpdateFailed` (not raw exceptions) from `_async_update_data`.
- Write helpers call `write_register` then `async_request_refresh`.

## modbus.py
- Uses `asyncio.open_connection` — no external Modbus library.
- Register addresses are 1-based (as in C4 manual); PDU sends `addr - 1`.
- All frame construction via `struct.pack`.

## Naming conventions
- Domain: `kvent`
- Unique ID pattern: `{entry_id}_{entity_key}`
- Device identifiers: `{(DOMAIN, entry_id)}`

## Tests
- `tests/test_modbus.py` — pure unit tests, no HA fixtures needed.
- `tests/test_fan.py` — fan entity tests with mocked coordinator.
- Mock Modbus at the `KVentModbusClient` level, not at socket level.

## Deploy
- Credentials come from `.env.ha` (gitignored); never hardcode.
- PowerShell: no `&&` — use separate lines or `;`.
- WSL key fix handled by `deploy-ha-wsl-bootstrap.sh`.

## Register map (Komfovent C4, 1-based)
- 1000  status      R/W  power on/off
- 1001  season      R/W  0=summer 1=winter
- 1007  warnings    R    binary; bit14 (0x4000) = service required
- 1100  speed_manual R/W write 1–3 only; read-back commanded level (+ OVR context)
- 1101  speed       R    actual 0–4 (standby / 1–3 / override)
- 1102  mode        R/W  0=manual 1=auto
- 1111  OVR enable  R/W  Boost preset uses 1
- 1112  OVR time    R/W  minutes 1–90
- 1200  supply_temp R    signed int16 / 10 → °C
- 1201  setpoint    R/W  tenths °C; doc 0..300 (0–30 °C)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxlarin63)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maxlarin63)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

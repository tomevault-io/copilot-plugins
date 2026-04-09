## acond-heat-pump

> Python library for Acond heat pumps via Modbus TCP. Tested on Acond PRO N only.

# Acond Heat Pump Library

Python library for Acond heat pumps via Modbus TCP. Tested on Acond PRO N only.

## Project Structure

```
acond_heat_pump/
  __init__.py          # Public API re-exports
  heat_pump.py         # AcondHeatPump — connect, read_data, all setters
  heat_pump_data.py    # HeatPumpResponse (24 registers) and HeatPumpStatus (13 status bits)
  constants.py         # HeatPumpMode and RegulationMode enums
  exceptions.py        # HeatPumpError, HeatPumpConnectionError
tests/
  test_heat_pump.py    # 31 unit tests (mocked pymodbus client)
```

## Key Architecture

- **Synchronous** library — all calls are blocking (Modbus TCP via pymodbus)
- Single connection per `AcondHeatPump(host, port=502)` instance
- `read_data()` reads 24 input registers (30001–30024) in one call, returns `HeatPumpResponse`
- Setters write to holding registers (40001+) individually

## Register Layout

- Registers 0–5: indoor1/2 temps (set/actual), DHW temps (set/actual)
- Register 6: status bits (13 bools packed in one u16)
- Registers 7–12: water back, outdoor, solar, pool temps, pool setpoint
- Registers 13–14: heat_pump_mode, regulation_mode (enums)
- Register 15: brine temp
- Register 16: heartbeat counter
- Registers 17–18: water outlet temp (actual/set)
- Registers 19–23: compressor capacity (max/actual), error codes (3)

## Temperature Read Validation

`_read_temp_register(value, min, max)` returns `None` if the converted value is outside `[min, max]`. This filters garbage register values but **read ranges must be wider than write ranges** — the pump may hold values set via its own interface that exceed the library's write limits. Lesson learned: DHW was capped at 46°C for reads which caused `None` when the pump had 50°C set.

## Write Limits (enforced in setter methods)

| Method | Register | Range |
|--------|----------|-------|
| `set_indoor_temperature(temp, circuit=1\|2)` | 40001/40003 | 10–30°C |
| `set_dhw_temperature(temp)` | 40005 | 10–50°C |
| `set_water_back_temperature(temp)` | 40008 | 10–65°C |
| `set_pool_temperature(temp)` | 40012 | 10–50°C |
| `set_water_cool_temperature(temp)` | 40013 | 15–30°C |

## Bitmask Operations

`change_setting()` and `set_summer_mode()` use read-modify-write on the TC_set register (40006):
- Mode bits 0–5 are mutually exclusive (cleared then one set)
- Bit 8 is summer mode (toggled independently)
- Non-mode bits (6–7, 9+) are preserved

## Testing

```sh
python -m pytest tests/ -v
```

All tests mock `pymodbus.client.ModbusTcpClient`. No real hardware needed.

## Conventions

- Version in `pyproject.toml` (Poetry)
- `CHANGELOG.md` follows Keep a Changelog format
- pymodbus 3.8+: use `device_id=` (not deprecated `slave=`), `count=` as keyword arg
- Temperature scaling: `round(temp * 10)` for writes, `value / 10.0` for reads

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jiribires)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jiribires)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## google-home-blade-mcp

> Google Home / Nest device control — thermostats, cameras, doorbells, event streams via SDM API


# Google Home Blade MCP — Skill Guide

## Token Efficiency Rules (MANDATORY)

1. **Use `ghome_status` for dashboard views** — one compact line per device, all key metrics
2. **Use `ghome_thermostats` for HVAC overview** — all thermostats in one call
3. **Use `ghome_devices` with type filter** — `device_type="THERMOSTAT"` avoids full list scan
4. **Use `ghome_device` only for full trait detail** — the compact views cover most needs
5. **Use `ghome_events` for recent activity** — pulls and acknowledges from Pub/Sub
6. **Never poll in a loop** — SDM rate limit is 5 commands/min per device

## Quick Start — 5 Most Common Operations

```
ghome_status                                      → Dashboard: all devices, one line each
ghome_thermostats                                 → All thermostats: ambient, setpoint, mode
ghome_thermostat_setpoint device="Living Room" heat_celsius=21.5  → Set temperature
ghome_thermostat_mode device="Living Room" mode="HEAT"            → Change mode
ghome_device device="Front Door"                  → Full camera/doorbell detail
```

## Tool Reference

### Meta
- **ghome_info** — Health check: API status, device counts, write gate, Pub/Sub status.

### Structure
- **ghome_structures** — List homes with IDs.
- **ghome_rooms** — List rooms in a structure.
- **ghome_devices** — List devices. Filter by type: THERMOSTAT, CAMERA, DOORBELL, DISPLAY.
- **ghome_device** — Full detail for one device: all traits, capabilities, status.

### Thermostat (write-gated)
- **ghome_thermostat_mode** — Set mode: HEAT, COOL, HEATCOOL, OFF.
- **ghome_thermostat_setpoint** — Set target temperature. Provide heat, cool, or both for range.
- **ghome_thermostat_eco** — Enable/disable eco mode: MANUAL_ECO or OFF.

### Fan (write-gated)
- **ghome_fan_set** — Fan timer: ON with duration, or OFF.

### Camera (write-gated)
- **ghome_camera_stream** — Generate live stream URL (WebRTC or RTSP). 5-min sessions.
- **ghome_camera_image** — Get snapshot from a camera event.

### Generic (write + confirm gated)
- **ghome_command** — Execute any SDM command. Requires `confirm=true`. Escape hatch.

### Events
- **ghome_events** — Pull recent events from Pub/Sub (motion, person, doorbell, state changes).

### Convenience
- **ghome_status** — Compact dashboard: all devices, one line each with key metrics.
- **ghome_thermostats** — All thermostats at a glance.

## Workflow Examples

### Morning Comfort Check
```
1. ghome_thermostats
   → See all thermostat states: ambient vs setpoint, HVAC running?
2. ghome_thermostat_setpoint device="Master Bedroom" heat_celsius=22.0
   → Warm up bedroom (if write enabled)
```

### Security Check
```
1. ghome_status
   → All devices: which cameras are online, any offline devices?
2. ghome_events max_messages=25
   → Recent motion/person/doorbell events
3. ghome_camera_image device="Front Door" event_id="abc123"
   → Get snapshot from doorbell event
```

### Energy Management
```
1. ghome_thermostats
   → Current state of all thermostats
2. ghome_thermostat_eco device="Living Room" eco_mode="MANUAL_ECO"
   → Enable eco mode for energy savings
3. ghome_thermostat_eco device="Office" eco_mode="MANUAL_ECO"
   → Same for office
```

### Full Device Inspection
```
1. ghome_structures
   → List homes
2. ghome_rooms structure_id="abc123"
   → Rooms in that home
3. ghome_devices device_type="CAMERA"
   → All cameras
4. ghome_device device="Garden Camera"
   → Full trait detail including stream protocols, event capabilities
```

## Common Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `device` | Device name or ID | `device="Living Room"` |
| `device_type` | Type filter | `device_type="THERMOSTAT"` |
| `structure_id` | Structure ID from ghome_structures | `structure_id="abc123"` |
| `mode` | Thermostat mode | `mode="HEAT"` |
| `heat_celsius` | Heat setpoint | `heat_celsius=21.5` |
| `cool_celsius` | Cool setpoint | `cool_celsius=25.0` |
| `eco_mode` | Eco setting | `eco_mode="MANUAL_ECO"` |
| `confirm` | Safety gate for raw commands | `confirm=true` |

## Output Format

Devices use compact pipe-delimited format:

```
Living Room | Thermostat | room=Living Room | online | 21.5°C | mode=HEAT | hvac=HEATING | id=abc123
Front Door | Doorbell | room=Entry | online | events=motion,person,chime | id=def456
```

Thermostat summary:
```
Living Room | Living Room | ambient=21.5°C | humidity=45% | mode=HEAT | heat→22.0°C | hvac=HEATING | online | id=abc123
```

## Security Notes

- All device commands blocked unless `GOOGLE_HOME_WRITE_ENABLED=true`
- `ghome_command` additionally requires `confirm=true` (double gate)
- Camera stream/image generation requires write gate (generates authenticated URLs)
- OAuth tokens never appear in output (credential scrubbing)
- Access tokens are memory-only — never written to disk
- Refresh token must be stored securely (env var with `secret: true`)
- SDM rate limit: 5 commands/min per device — do not poll

## Rate Limits

| Operation | Limit |
|-----------|-------|
| Device commands | 5/min per device |
| API aggregate | 6,000/60s per project |
| Camera streams | 5-min sessions |
| Sandbox users | 25 max |

---
> Source: [Groupthink-dev/google-home-blade-mcp](https://github.com/Groupthink-dev/google-home-blade-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

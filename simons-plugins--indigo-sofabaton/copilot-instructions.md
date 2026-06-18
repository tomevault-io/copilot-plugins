## indigo-sofabaton

> - **Bump `PluginVersion`** in `Sofabaton.indigoPlugin/Contents/Info.plist` with every PR — CI enforces this

# CLAUDE.md

## Release Process

- **Bump `PluginVersion`** in `Sofabaton.indigoPlugin/Contents/Info.plist` with every PR — CI enforces this
- Version format: `YYYY.R.P` (year.release.patch), e.g. `2026.0.2`
- A version-check workflow blocks PRs that reuse an existing tag
- On merge to main, the create-release workflow auto-creates a tagged GitHub release with the `.indigoPlugin.zip` bundle

## Plugin Overview

**Sofabaton** - Indigo plugin for Sofabaton X2 and X1/X1S universal remote hubs

- **Version**: 2026.1.0
- **Bundle ID**: `com.simons-plugins.sofabaton`
- **Protocols**: MQTT (X2 via paho-mqtt 2.1), TCP (X1/X1S via binary protocol)
- **Hub Models**: Sofabaton X2 (MQTT), X1/X1S (TCP)

Controls and monitors Sofabaton hubs. The X2 connects via MQTT (configure via Sofabaton app → Home Assistant). The X1/X1S connects directly via TCP using a proprietary binary protocol with auto-discovery via mDNS.

## Architecture

### Communication

- **X2 Transport**: MQTT via paho-mqtt library (bundled in Packages/)
- **X1/X1S Transport**: TCP binary protocol via `transport_tcp.py`
- **Discovery**: mDNS via zeroconf — `_sofabaton_hub._udp.local.` (X2), `_x1hub._udp.local.` (X1/X1S)
- **X2 Topics**: `activity/{MAC}/*` and `device/{MAC}/*` where MAC is 12-char uppercase hex
- **X2 Payloads**: JSON
- **X1/X1S Protocol**: Binary frames with sync bytes, opcodes, and checksums (see `protocol_const.py`, `frame_codec.py`, `opcode_handlers.py`)
- **Hub constraint**: Single-threaded processing, 200ms delay between publishes

### Device Types

| Type ID | Indigo Type | Description |
|---------|-------------|-------------|
| `sofabatonHub` | custom | X2 hub — shows connection status and active activity (MQTT) |
| `sofabatonX1Hub` | custom | X1/X1S hub — shows connection status, active activity, device count (TCP) |
| `sofabatonActivity` | relay | One per activity — on/off maps to activity active/inactive |

Activity devices are auto-created when the plugin discovers activities from either hub. The plugin routes commands to the correct transport (MQTT or TCP) based on which hub the activity belongs to.

### X1/X1S TCP Protocol Modules

| Module | Description |
|--------|-------------|
| `protocol_const.py` | Sync bytes, opcodes, port numbers, mDNS service types |
| `frame_codec.py` | Build/parse binary frames, extract frames from byte stream |
| `opcode_handlers.py` | Parse device/activity catalog rows and button records |
| `transport_tcp.py` | TCP client with reconnect, catalog sync, activity/device commands |

### MQTT Topics Used (X2 only)

**Subscribe (receive from hub):**
- `activity/{mac}/list` — Activity list response
- `activity/{mac}/activity_control_up` — Activity state changes (from remote/app)
- `activity/{mac}/keys_list` — Assigned key list
- `activity/{mac}/macro_keys_list` — Macro key list
- `activity/{mac}/favorites_keys_list` — Favorite key list
- `device/{mac}/list` — Device list response
- `device/{mac}/keys_list` — Device key list

**Publish (send to hub):**
- `activity/{mac}/list_request` — Request activity list
- `activity/{mac}/activity_control_down` — Activate/deactivate activity
- `activity/{mac}/keys_control` — Send key press
- `activity/{mac}/macro_keys_control` — Send macro key
- `activity/{mac}/favorites_keys_control` — Send favorite key

### Key IDs (27 hardware remote keys)

| Key | ID | Key | ID | Key | ID |
|-----|----|-----|----|-----|----|
| up | 174 | volume_up | 182 | rewind | 187 |
| down | 178 | volume_down | 185 | play | 156 |
| left | 175 | mute | 184 | fast_forward | 189 |
| right | 177 | channel_up | 183 | pause | 188 |
| ok | 176 | channel_down | 186 | dvr | 155 |
| back | 179 | guide | 157 | exit | 154 |
| home | 180 | red | 190 | a | 153 |
| menu | 181 | green | 191 | b | 152 |
| | | yellow | 192 | c | 151 |
| | | blue | 193 | | |

### Firmware Quirk

Favorite key control uses `device_id` in the `activity_id` field (not the actual activity_id). This is a known Sofabaton firmware design issue.

## Prerequisites

**For X2 hub:**
1. **MQTT Broker** — Mosquitto, or Indigo's MQTT Broker plugin by FlyingDiver
2. **Sofabaton X2 hub** configured to connect to your MQTT broker (via Sofabaton app → Add Device → Home Assistant)

**For X1/X1S hub:**
1. **Sofabaton X1 or X1S hub** on the same network — discovered automatically via mDNS or configured with IP address

## Available Actions

| Action | Description |
|--------|-------------|
| Turn On/Off activity | Activate or deactivate a Sofabaton activity (X2 or X1) |
| Send Key Press | Send any of the 27 remote keys to an activity (X2) |
| Send Macro Key | Trigger a macro by key ID (X2) |
| Send Favorite Key | Trigger a favorite by key ID + device ID (X2) |
| Send Device Command | Send a command to an X1/X1S device (X1 only) |
| Refresh Activities | Re-query hub for activity list |
| Stop All Activities | Turn off all active activities |

## Menu Items

| Menu Item | Description |
|-----------|-------------|
| Discover Hub (mDNS) | Find Sofabaton hubs on the network |
| Refresh Activities | Query hub for current activities |
| List Macro Keys | Log all macro keys for all activities |
| List Favorite Keys | Log all favorite keys for all activities |

## Testing

```bash
# Copy to Indigo server
cp -r "Sofabaton.indigoPlugin" "/Volumes/Macintosh HD-1/Library/Application Support/Perceptive Automation/Indigo 2025.1/Plugins/"
```

Enable via Indigo: Plugins → Manage Plugins → Enable Sofabaton

## Dependencies (bundled)

- `paho-mqtt` 2.1.0 — MQTT client
- `zeroconf` 0.148.0 — mDNS discovery
- `ifaddr` 0.2.0 — Network interface addresses (zeroconf dependency)

---
> Source: [simons-plugins/indigo-sofabaton](https://github.com/simons-plugins/indigo-sofabaton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

## heatmiser

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**Heatmiser Neo** - Indigo plugin for Heatmiser Neo thermostats and smart plugs

- **Version**: 2023.1.0 (API 3.0 / Python 3)
- **Bundle ID**: `com.racarter.indigoplugin.heatmiser-neo`
- **Author**: Alan Carter
- **Documentation**: https://carter53.wordpress.com/indigo/heatmiser/

Integrates Heatmiser Neo heating system with Indigo home automation via the Neohub central controller.

## Versioning & Release

### Version bump is required for every PR

The `PluginVersion` in `HeatmiserNeo.IndigoPlugin/Contents/Info.plist` must be bumped in every PR. CI runs a version-check that fails if the version already exists as a git tag. **Do not merge with failing checks.**

Version format: `YYYY.R.patch` (e.g. `2026.0.2`). Bump the patch for fixes/docs, minor for features.

On merge to main, the `create-release` workflow automatically creates a GitHub release with a `.zip` bundle of the plugin.

### PR checklist

1. Bump `PluginVersion` in `Info.plist`
2. Push and create PR
3. Wait for version-check CI to pass
4. Merge only after all checks are green

## Architecture

### Communication Protocol

The plugin supports two connection modes to the **Neohub** device:

**WSS — Secure WebSocket (default, recommended):**
- **Port**: 4243 (TLS, self-signed cert)
- **Protocol**: Two-layer JSON envelope over WebSocket. Outer message has `message_type` + `message` fields; inner payload contains API token and `COMMANDS` array with `COMMAND`/`COMMANDID` pairs.
- **Authentication**: API token (generated in Heatmiser Neo App → Settings → Api Access → Tokens)
- **Connection**: Fresh WebSocket opened per command (closed in `finally`). Thread-safe via `_wss_lock`. An earlier persistent-connection design was abandoned after Indigo 2025.2 / Python 3.13 / `websockets` 15.x began raising `ProtocolError("incorrect masking")` on long-lived sockets — the per-call model trades one TLS handshake per poll for immunity to that class of state-desync bug.
- **Timeout**: 8 seconds for connection, 10 seconds for response

**Legacy TCP (port 4242):**
- **Port**: 4242 (unencrypted, no auth)
- **Protocol**: JSON over TCP with null terminator (`\0`). Opens a new socket per command.
- **Timeout**: 8 seconds for socket operations

- **IP Address**: Configurable via plugin preferences (default: 192.168.0.1)

All commands are sent as JSON objects: `{"COMMAND":value}`. The WSS path wraps them in the token-authenticated envelope; the TCP path sends them directly with a null terminator.

### Concurrent Thread Architecture

The plugin runs a continuous monitoring thread ([plugin.py:84-108](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L84-L108)) that:

1. **Every 30 seconds**: Polls Neohub for device status updates via `INFO` command
2. **Once daily at specific times**:
   - 03:00 - Synchronizes Neohub time/date with Indigo (if enabled)
   - 00:00 - Fetches DCB (Device Control Block) data
   - 05:00 - Fetches Engineering data
3. **On first run**: Configures NTP settings based on time sync preference

**Thread Sleep Pattern**: 1 second poll + 29 second wait = 30 second cycle

### Device Auto-Discovery

On startup, the plugin queries the Neohub for all connected devices ([plugin.py:144-186](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L144-L186)):

1. Sends `{"INFO":0}` command to discover devices
2. Creates Indigo device for each physical device (if not already created)
3. Uses the device's array index in the Neohub as the Indigo device address
4. Automatically determines device type and creates appropriate Indigo device

**Device Naming Restriction**: Device names from Heatmiser must be valid Python identifiers (no spaces, special characters except underscore). The plugin validates this with `isidentifier()` and logs an error if invalid.

### Supported Device Types

| Device Type | Description | Indigo Device Type |
|-------------|-------------|-------------------|
| 1 | NeoStat thermostat | `heatmiserNeostat` |
| 6 | NeoPlug smart plug | `heatmiserNeoplug` |
| 7 | NeoAir thermostat | `heatmiserNeostat` |
| 12 | NeoStat-e thermostat | `heatmiserNeostat` |
| 13 | NeoAir (newer model) | `heatmiserNeostat` |
| 14 | Wireless air sensor | `heatmiserNeoSensor` |
| 0 | Offline device | (sets error state) |

### State Management

**NeoStat Thermostats** track:
- Current temperature, setpoint, heat state
- Pre-heat mode, frost protection (mapped to "Cool" mode)
- Away/Holiday modes
- Hold time remaining and hold temperature
- Window open, low battery, keypad locked
- Floor temperature (when probe present, 127 = no probe)
- Hub firmware version, DST/NTP status (stored on first device)
- Engineering data: Rate of Change, Frost Temp, Switching Differential

**NeoTimeclock** tracks:
- On/Off state, temperature, short mode
- Away/Holiday modes
- Hold time remaining (for timer boost)

**NeoPlug** tracks:
- On/Off state based on timer status

**Wireless Air Sensors** track:
- Current temperature reading
- Sensor valid/online status
- Low battery warning

### Mode Mapping

Heatmiser modes are mapped to Indigo HVAC modes:

| Heatmiser Mode | Indigo Mode | Description |
|----------------|-------------|-------------|
| Normal schedule | ProgramHeat | Following programmed schedule |
| Frost protection (STANDBY) | Cool | Building protection mode |
| Temperature hold (TEMP_HOLD) | Heat | Manual override/boost |

## Plugin Configuration

**PluginConfig.xml** defines these settings:

1. **neohubIP**: IP address of Neohub (default: 192.168.0.1)
2. **connectionMode**: `wss` (default) or `tcp` — selects WSS port 4243 or legacy TCP port 4242
3. **neohubToken**: API token for WSS authentication (only visible when connectionMode is `wss`)
4. **neohubGen2**: Gen 2 API toggle (only visible when connectionMode is `tcp`; forced `True` for WSS)
5. **timeSync**: If true, sync Neohub time with Indigo daily; if false, use NTP
6. **logComms**: If true, log all communications to Indigo Event Log (token is redacted in WSS logs)

## Available Actions

Defined in [Actions.xml](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/Actions.xml):

1. **Heating Override** (`setOverride`): Temporarily override schedule
   - Duration: 30 minutes to 20 hours
   - Temperature: 10-25°C
2. **Building Protection** (`setCool`): Enable frost protection mode
3. **Auto Mode** (`setAuto`): Return to programmed schedule
4. **Timer Boost** (`timerBoost`): Boost timeclock device on for set duration
5. **Cancel Timer Boost** (`timerBoostOff`): Cancel active timer boost
6. **Set Holiday** (`setHoliday`): Set all devices to holiday mode until end date
7. **Cancel Holiday** (`cancelHoliday`): Cancel holiday mode
8. **Away Mode On** (`awayOn`): Set device to away mode (reduced temperature)
9. **Away Mode Off** (`awayOff`): Cancel away mode, restore schedule
10. **Lock Keypad** (`lockKeypad`): Lock thermostat keypad with 4-digit PIN
11. **Unlock Keypad** (`unlockKeypad`): Remove keypad lock
12. **Cancel Hold** (`cancelHold`): Cancel active temperature hold
13. **Set Frost Temp** (`setFrostTemp`): Set frost protection temperature per device
14. **Identify Device** (`identifyDevice`): Flash LED to locate a device
15. **Change Neohub IP** (`changeIp`): Update IP address without plugin restart

## Key Implementation Details

### Error Handling Strategy

The plugin uses error counters to avoid log spam:
- `connectErrorCount` / `sendErrorCount`: Track consecutive failures per transport
- **WSS path**: Logs the first 3 errors, then every 10th failure. Counters reset on success.
- **TCP path**: Logs only on the 3rd consecutive failure. `sendErrorCount` resets after logging.
- Both counters reset to 0 on any successful command.

### Multi-Packet Response Handling (TCP only)

On the legacy TCP path, some commands (INFO/GET_LIVE_DATA, ENGINEERS_DATA/GET_ENGINEERS) return large JSON responses that may arrive in multiple TCP packets. The plugin handles this by checking for complete JSON markers:
- `INFO`/`GET_LIVE_DATA` command: Waits for `}]}` at end of response
- `ENGINEERS_DATA`/`GET_ENGINEERS`: Waits for `}}` at end of response

The WSS path does not need this — WebSocket messages are framed at the protocol level.

### JSON Parsing Resilience

The plugin strips extraneous non-printable characters before JSON parsing ([plugin.py:298](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L298)):
```python
datak = re.sub(b'[^\s!-~]', b'', dataj)  # Filter characters outside printable ASCII range
```

### Device Superseding

Devices with "SUPERSEDED" in their name are ignored during discovery and updates ([plugin.py:158, 397](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L158)). This allows graceful device replacement without deleting old device records.

## Testing the Plugin

### Installation

```bash
# Copy plugin to Indigo plugins directory
cp -r "HeatmiserNeo.IndigoPlugin" "/Library/Application Support/Perceptive Automation/Indigo 2023.2/Plugins/"

# Or to disabled plugins folder for development
cp -r "HeatmiserNeo.IndigoPlugin" "/Library/Application Support/Perceptive Automation/Indigo 2023.2/Plugins (Disabled)/"
```

Then enable via: **Indigo → Plugins → Manage Plugins**

### Configuration

1. Enter your Neohub IP address in plugin preferences
2. Enable "Copy Neo comms to Indigo Event Log" for debugging
3. Optionally enable time sync if you want Indigo to manage Neohub time

### Debugging Commands

All Neohub commands are sent via `getNeoData()` method. Common commands:
- `{"INFO":0}` - Get all device status
- `{"READ_DCB":100}` - Get Device Control Block
- `{"ENGINEERS_DATA":0}` - Get engineering data
- `{"SET_TEMP":[20, "DeviceName"]}` - Set temperature
- `{"FROST_ON":["DeviceName"]}` - Enable frost protection
- `{"TIMER_ON":["DeviceName"]}` - Turn on NeoPlug

## Important Constraints

### Device Names
Heatmiser device names **must** be valid Python identifiers:
- ✅ `Living_Room`, `Bedroom1`, `Hall`
- ❌ `Living Room` (space), `Bed/Room` (slash), `Hall-way` (hyphen)

If invalid names are detected, the plugin logs an error and you must rename the device on the Neohub.

### Cooling Not Supported
Heatmiser Neo devices don't support cooling. The "Cool" mode in Indigo is mapped to frost protection. Indigo commands for cool setpoints are ignored.

### Status Update Frequency
Device status is updated every 30 seconds. Manual status request actions are not supported - the plugin logs "Status automatically updated every 30 seconds" for these requests.

## Development History

See [Versions.txt](HeatmiserNeo.IndigoPlugin/Versions.txt) for complete changelog. Key milestones:
- **2023.1.0** (Dec 2023): Added NeoAir support, superseded device handling
- **2022.1.0** (Aug 2022): Added Holiday Mode
- **2022.0.0** (Apr 2022): Updated for Indigo 2022.1+ and Python 3
- **0.5.0** (Jun 2019): Added 30-minute override option
- **0.3.0** (Oct 2017): Made time sync optional with NTP support

## Related Documentation

For general Indigo plugin development, see the parent [CLAUDE.md](../CLAUDE.md) which covers:
- Plugin lifecycle and structure
- Debugging techniques
- SDK examples and references
- Common development tasks

## Neohub API Reference

The plugin uses undocumented Heatmiser Neohub socket API. Key commands discovered through reverse engineering:

| Command | Purpose |
|---------|---------|
| `INFO` | Get all device status |
| `READ_DCB` | Get hub configuration (firmware, DST, NTP) |
| `ENGINEERS_DATA` | Get advanced device parameters |
| `SET_TEMP` | Set target temperature |
| `SET_TIME` / `SET_DATE` | Sync time/date |
| `FROST_ON` / `FROST_OFF` | Control building protection |
| `HOLD` | Temporary temperature override |
| `TIMER_ON` / `TIMER_OFF` | Control NeoPlug |
| `NTP_ON` / `NTP_OFF` | Enable/disable NTP |
| `DST_ON` / `DST_OFF` | Enable/disable DST |

---
> Source: [simons-plugins/heatmiser](https://github.com/simons-plugins/heatmiser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

## homeassistant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant configuration repository running version 2025.10.1. The setup includes:

- Main Home Assistant configuration with YAML-based config
- Custom ZHA (Zigbee Home Automation) device quirks
- ESPHome device configurations for M5Stack Atom Echo devices
- Custom components (HACS-managed)
- Automation blueprints for various lighting and sensor scenarios

## Architecture

### Core Configuration Structure
- `configuration.yaml` - Main HA configuration using include directives to organize all components
  - **Automations**: `automation: !include automations.yaml` - Single file containing ALL automations as a YAML list
  - MQTT lights/sensors loaded from `mqtt/` directory
  - ZHA setup with custom quirks enabled
  - Notification groups from `notify/groups.yaml`
  - Lovelace dashboards configured in YAML mode from `dashboards/` directory
  - Input helpers organized by type in `input_helpers/` subdirectories
- `automations.yaml` - **CRITICAL**: Single file containing all automations as a flat YAML list
  - All automations are stored directly in this file, NOT in subdirectories
  - Each automation is a list item starting with `- id: 'unique_id'`
  - Common automation types: Blueprint-based (Shelly i4 buttons, motion sensors), custom event-driven, time-based triggers
  - Heavy use of `use_blueprint` for consistency (Danieldz/shelly-i4-4-buttons-actions.yaml, Blackshome/sensor-light.yaml)
- `scripts/` - Scripts organized by category subdirectories (lighting, family, dashboard, device config, testing)
  - Loaded via `!include_dir_merge_named scripts/` in configuration.yaml
- `scenes/` - Scene definitions organized by room subdirectories (bathroom, living_room, dashboard)
  - Loaded via `!include_dir_merge_list scenes/` in configuration.yaml

### Custom Components (HACS-managed)
- `custom_components/family_safety/` - Microsoft Family Safety integration
- `custom_components/hacs/` - Home Assistant Community Store integration
- `custom_components/bermuda/` - Bluetooth proximity tracking
- `custom_components/circadian_lighting/` - Adaptive lighting based on time of day
- `custom_components/dreame_mower/` - Dreame robot mower integration
- `custom_components/ha_strava/` - Strava fitness tracking integration
- `custom_components/zaptec/` - Zaptec EV charger integration
- Custom components follow standard HA integration patterns with manifests, coordinators, and entity classes

### ZHA Quirks
- `custom_zha_quirks/hue_wall_switch_rdm004.py` - Custom device quirk for Philips/Signify RDM004 wall switches
- Implements custom clusters for handling button press events and device-specific behaviors

### ESPHome Configurations
- `esphome/` - Device configs for M5Stack Atom Echo voice assistants
- Uses packages from GitHub firmware repository
- Each device has unique encryption keys and WiFi credentials from secrets

### Blueprints
- `blueprints/automation/` - Community automation blueprints for motion lighting, wall switches, and notifications
- `blueprints/script/` - Script blueprints for device configuration and notifications
- `blueprints/template/` - Template blueprints for sensor inversions

## Development Commands

This is a configuration-based Home Assistant setup with no build/test/lint commands. Development involves:

- Edit YAML configuration files directly
- Use Home Assistant's built-in configuration validation
- Test automations and scripts through HA UI
- ESPHome configurations can be validated with `esphome config <file.yaml>`

## Git Workflow and Deployment

**CRITICAL REMINDER**: Claude Code should NEVER commit changes or update Home Assistant directly. The user handles all Git operations and Home Assistant updates.

⚠️ **IMPORTANT**: Claude CANNOT test changes against the live Home Assistant system until AFTER the user has committed and deployed the changes. All REST API calls and MCP operations only work AFTER user deployment.

**Claude's Role:**
- Make configuration file changes locally in the repository
- Validate configuration syntax when possible (YAML structure only)
- Provide recommendations and implementation
- **NEVER assume entity_IDs exist until user confirms after deployment**

**User's Role:**
- Git commit and push changes to repository
- Update Home Assistant from Git repository
- Test and verify changes in live Home Assistant environment
- Report back any validation errors to Claude for fixes

**Deployment Process:**
1. Claude makes local file changes
2. User commits and pushes to Git
3. User updates Home Assistant from Git
4. User tests configuration validation
5. User reports any errors back to Claude for fixes

This separation ensures proper change control and prevents Claude from making assumptions about live system state before deployment.

## MCP Integration

This repository has Model Context Protocol (MCP) integration set up for direct Home Assistant access:

- **Home Assistant MCP Server**: Installed and running at `http://192.168.1.191:8123/mcp_server/sse`
- **Access Token**: Long-lived token stored in `.env` file (HA_TOKEN)
- **Local Proxy**: mcp-proxy installed via `uv tool install git+https://github.com/sparfenyuk/mcp-proxy`
- **Claude Code Configuration**: MCP server configured in `~/.claude.json` for automatic integration

### MCP Server Status
✅ **WORKING** - MCP integration is functional and configured:
- **Server**: home-assistant version 1.5.0
- **Connection**: HTTP 200 OK via mcp-proxy
- **Capabilities**: Tools and Prompts available
- **Authentication**: Bearer token authentication working

### Testing MCP Connection
```bash
source ~/.local/bin/env && mcp-proxy -H Authorization "Bearer $HA_TOKEN" http://192.168.1.191:8123/mcp_server/sse --debug
```

### Available MCP Methods

Claude Code can interact directly with Home Assistant through these MCP tools:

#### Device Control
- **`mcp__home-assistant__HassTurnOn`** - Turn on devices (lights, switches, etc.)
  ```
  mcp__home-assistant__HassTurnOn(name="Lampnamn", area="Område", floor="Våning")
  ```
- **`mcp__home-assistant__HassTurnOff`** - Turn off devices
  ```
  mcp__home-assistant__HassTurnOff(name="Lampnamn", area="Område")
  ```

#### Lighting Control
- **`mcp__home-assistant__HassLightSet`** - Set light brightness, color, temperature
  ```
  mcp__home-assistant__HassLightSet(name="Lampnamn", brightness=75, color="red", temperature=3000)
  ```

#### Media Player Control
- **`mcp__home-assistant__HassMediaPause`** - Pause media players
- **`mcp__home-assistant__HassMediaUnpause`** - Resume media players
- **`mcp__home-assistant__HassMediaNext`** - Skip to next track
- **`mcp__home-assistant__HassMediaPrevious`** - Go to previous track
- **`mcp__home-assistant__HassSetVolume`** - Set volume level (0-100)
- **`mcp__home-assistant__HassMediaSearchAndPlay`** - Search and play media

#### Climate Control
- **`mcp__home-assistant__HassClimateSetTemperature`** - Set target temperature
  ```
  mcp__home-assistant__HassClimateSetTemperature(name="Värmepump", temperature=22)
  ```

#### Todo Lists
- **`mcp__home-assistant__HassListAddItem`** - Add item to todo list
- **`mcp__home-assistant__HassListCompleteItem`** - Mark todo item as complete
- **`mcp__home-assistant__todo_get_items`** - Get items from todo lists

#### System Information
- **`mcp__home-assistant__GetLiveContext`** - Get current state of all devices and areas
  ```
  mcp__home-assistant__GetLiveContext()
  ```

#### Communication
- **`mcp__home-assistant__HassBroadcast`** - Broadcast message through home speakers
- **`mcp__home-assistant__HassCancelAllTimers`** - Cancel all running timers

### MCP Usage Guidelines
1. **Always use MCP methods first** for device control and status checking
2. **Prefer specific device names** over areas when possible for precision
3. **Check GetLiveContext** to understand current device states before making changes
4. **Use Swedish device names** as they appear in Home Assistant
5. **Test device availability** by attempting control rather than relying on "unavailable" status

This enables Claude Code to:
- Read entity states and configurations in real-time
- Control all smart home devices directly
- Test automations and scripts immediately
- Diagnose connectivity issues by attempting device control
- Update configurations and verify functionality instantly

## Key Patterns

### File Organization Strategy
This repository uses a modular structure with include directives:
- **Automation organization**: All automations stored in single `automations.yaml` file
  - Each automation is a list item: `- id: 'unique_id'`
  - When adding new automations, append to `automations.yaml` maintaining list format
  - Do NOT create separate automation files in subdirectories
- **Script organization**: Scripts split into category subdirectories under `scripts/`
  - Loaded via `!include_dir_merge_named scripts/`
  - Each script file contains named script definitions (no leading dash)
- **Scene organization**: Scenes split into room subdirectories under `scenes/`
  - Loaded via `!include_dir_merge_list scenes/`
  - Each scene file MUST start with `- id:` (list format)
- **Input helper organization**: Organized by type in `input_helpers/` subdirectories
  - `input_helpers/input_boolean/` - Boolean toggles (lighting, dashboard, family settings)
  - `input_helpers/input_number/` - Numeric values (brightness, screen time, fan speed)
  - `input_helpers/input_button/` - Trigger buttons (family timers, screen time requests)
  - `input_helpers/timer/` - Timer entities (family countdown timers)
  - Loaded via `!include_dir_merge_named` for each type
- **Light Groups**: All managed via GUI (Settings > Helpers), NOT in `light_groups.yaml`
  - `light_groups.yaml` is kept empty with comments only
  - Use GUI groups: `light.grupp_*` (e.g., `light.grupp_matsal`, `light.grupp_arbetsrum`)
- **YAML dashboards**: Stored in `dashboards/` and configured via `lovelace:` in configuration.yaml

### Lighting Scripts
- Sequential lighting effects with staggered delays (e.g., `grus_trip_trapp_trull`)
- Parallel execution with timed sequences for synchronized wave effects (e.g., `toalett_belysning_on`)
- Consistent transition times (1-3s) and brightness levels (typically 50-60%)
- Scripts organized in `scripts/lighting/sequential_effects.yaml`

### MQTT Integration
- Custom LED controls for Pi GPIO and Pico W devices
- Configuration split into `mqtt/lights.yaml` and `mqtt/binary_sensors.yaml`
- Standardized topic structure: `homeassistant/device/component/action`
- QoS 1 and retain flags for reliability

### Automation Structure
- **Blueprint-based automations** are heavily used for consistency:
  - `Danieldz/shelly-i4-4-buttons-actions.yaml` - Shelly i4 button controls
    - Supports 4 buttons with btn_down/btn_up events
    - Uses `light_target_bN` inputs for automatic light toggle functionality on btn_up
    - Supports custom actions via downbN/upbN/singlebN/doublebN/longbN inputs
    - Mode: restart with silent max_exceeded
  - `Blackshome/sensor-light.yaml` - Motion/sensor-triggered lighting
    - Configurable delays (typically 2.5-5 minutes), transitions, brightness, color temp
    - Supports dynamic lighting based on sun elevation
  - `cfeenstra/philips-hue-wall-module-rdm001-rdm004-button-press-blueprint.yaml` - Philips Hue wall switches
- **Custom automations** for complex logic:
  - Air quality monitoring with multi-stage alerts and rate limiting
  - Screen time tracking with Microsoft Family Safety integration
  - Family dinner timer with LED notifications and MQTT display updates
  - Media player-based scene activation (Chromecast state → lighting scenes)
- **Naming conventions**: Automations use Swedish names/aliases for user-facing elements
- **Device/Entity references**: Mix of `device_id`, `entity_id`, and `area_id` depending on use case

### ZHA Quirks Development
- Custom cluster implementations for unsupported device features
- Event-based button handling with proper press type detection
- Device signature matching for automatic quirk application
- Quirks placed in `custom_zha_quirks/` directory and loaded via `zha.custom_quirks_path`

---
> Source: [BloomBoy/homeassistant](https://github.com/BloomBoy/homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->

## hass-beeminder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Since this is a Home Assistant custom component, development follows standard Home Assistant patterns:

```bash
# Install Home Assistant development environment
pip install homeassistant

# Run Home Assistant with this custom component
hass -c /path/to/config/dir

# Check Home Assistant logs for integration errors
tail -f /path/to/config/dir/home-assistant.log
```

## Architecture Overview

This is a custom Home Assistant integration for Beeminder that creates sensors to track goal progress.

### Core Components

1. **Data Coordinator** (`__init__.py`): 
   - `BeeminderDataUpdateCoordinator` fetches data from Beeminder API every 5 minutes
   - Manages shared state for all sensors to minimize API calls
   - Uses Home Assistant's `DataUpdateCoordinator` pattern

2. **Sensor Implementation** (`sensor.py`):
   - Creates three sensors per Beeminder goal:
     - Current value sensor: tracks the current cumulative value
     - Goal value sensor: tracks the dynamic target value
     - Days until derailment sensor: tracks how many days until the goal derails
   - All sensors expose comprehensive goal data as attributes including:
     - All Beeminder API fields (pledge, tags, goal type, urgency, etc.)
     - Last 100 datapoints with timestamps, values, comments, and IDs
     - Derailment tracking with `is_derailing_soon` and `is_derailing_today` flags

3. **Configuration** (supports two methods):
   - YAML configuration via `configuration.yaml` (primary)
   - UI configuration via config flow (basic implementation)

### Key Integration Patterns

- **API Flow**: Beeminder API → DataUpdateCoordinator → Sensor Entities → Home Assistant State
- **Update Pattern**: Coordinator polls → Updates internal data → Sensors automatically reflect changes
- **Error Handling**: API failures raise `UpdateFailed`, coordinator handles retry logic

### Beeminder API Usage

- Base URL: `https://www.beeminder.com/api/v1/users/{username}`
- Authentication: `auth_token` parameter
- Endpoints:
  - `/goals.json` - Fetch all goals
  - `/goals/{slug}/datapoints.json?count=100&sort=daystamp` - Fetch goal datapoints

### Development Considerations

- Always use async/await patterns (Home Assistant requirement)
- Sensor entities must extend both `CoordinatorEntity` and `SensorEntity`
- Use `_attr_` prefixed properties for entity attributes
- Store sensitive data (auth tokens) in `secrets.yaml`
- Integration restarts required after code changes

### Example Automations

To create warnings for goals due today, use automations like:

```yaml
# Alert when any goal is derailing today
automation:
  - alias: "Beeminder Derailment Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.beeminder_*_days_until_derailment
        below: 1
    action:
      - service: notify.notify
        data_template:
          title: "Beeminder Alert!"
          message: "Goal {{ trigger.to_state.attributes.title }} is derailing today!"

# Dashboard card for goals due today
type: entity-filter
entities:
  - sensor.beeminder_*_days_until_derailment
state_filter:
  - operator: '<'
    value: 1
card:
  type: entities
  title: Goals Due Today
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wstewarttennes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wstewarttennes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

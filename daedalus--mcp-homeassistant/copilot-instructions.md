## mcp-homeassistant

> MCP server exposing Home Assistant REST API functionality.

# MCP Home Assistant

MCP server exposing Home Assistant REST API functionality.

## When to use this skill

Use this skill when you need to:
- Control Home Assistant entities
- Get entity states
- Call Home Assistant services
- Query historical data
- Fire custom events

## Tools

- `get_api_status` - Check if API is running
- `get_config` - Get HA configuration
- `get_components` - List loaded components
- `get_events` - List event types
- `get_services` - List services by domain
- `get_states` / `get_state` - Get entity states
- `set_state` / `delete_state` - Update states
- `fire_event` - Fire custom event
- `call_service` - Call HA service
- `render_template` - Render Jinja template
- `get_history` - Get historical data
- `get_logbook` - Get logbook entries
- `check_config` - Validate configuration

## Install

```bash
pip install mcp-homeassistant
```

Requires: `HA_URL` and `HA_TOKEN` environment variables

---
> Source: [daedalus/mcp-homeassistant](https://github.com/daedalus/mcp-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

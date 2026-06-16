## mcp-kicad

> MCP server exposing KiCad PCB Editor functionality via IPC API.

# MCP KiCad

MCP server exposing KiCad PCB Editor functionality via IPC API.

## When to use this skill

Use this skill when you need to:
- Interact with KiCad PCB designs
- Query board information
- List footprints, nets, components
- Inspect PCB design data

## Tools

- **Connection**: `connect_kicad`, `check_connection`
- **Board**: `get_board_info`, `get_layers`, `get_board_design_settings`
- **Footprints**: `list_footprints`, `get_footprint`, `search_footprints`
- **Nets**: `list_nets`, `get_net`, `list_netclasses`
- **Zones**: `list_zones`
- **Tracks/Vias**: `list_tracks`, `list_vias`
- **Pads**: `list_pads`
- **Text**: `list_texts`, `get_text_extents`
- **Project**: `get_project`, `get_version`

## Requirements

KiCad 9.0+ with IPC API server enabled (Preferences > Plugins)

## Install

```bash
pip install mcp-kicad
```

---
> Source: [daedalus/mcp-kicad](https://github.com/daedalus/mcp-kicad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

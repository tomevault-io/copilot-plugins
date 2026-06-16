## mcp-snap7

> MCP server for python-snap7, enabling interaction with Siemens PLCs.

# MCP Snap7

MCP server for python-snap7, enabling interaction with Siemens PLCs.

## When to use this skill

Use this skill when you need to:
- Connect to Siemens PLCs
- Read/write data blocks
- Read/write memory
- Control PLC execution

## Tools

- `connect_plc` / `disconnect_plc` - Connection
- `get_connected` - Connection status
- `db_read` / `db_write` - Data blocks
- `mb_read` / `mb_write` - Memory bytes
- `tm_read` / `tm_write` - Timers
- `ct_read` / `ct_write` - Counters
- `eb_read` / `eb_write` - Edge inputs
- `ab_read` / `ab_write` - Absolute bytes
- `get_cpu_info` / `get_cpu_state` - CPU info
- `get_protection` - Protection level
- `plc_cold_start` / `plc_hot_start` / `plc_stop` - PLC control
- `get_error_text` - Error descriptions

## Install

```bash
pip install mcp-snap7
```

Requires: libsnap7 library on system

---
> Source: [daedalus/mcp-snap7](https://github.com/daedalus/mcp-snap7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

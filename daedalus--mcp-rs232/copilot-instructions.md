## mcp-rs232

> MCP server exposing RS232 serial port connectivity.

# MCP RS232

MCP server exposing RS232 serial port connectivity.

## When to use this skill

Use this skill when you need to:
- Communicate with serial devices
- Read/write to COM ports
- List available serial ports

## Tools

- `list_ports_tool` - List available serial ports
- `open_port` - Open serial connection
- `close_port` - Close connection
- `read_port` - Read data (hex encoded)
- `write_port` - Write hex-encoded data
- `get_port_config` - Get port configuration

## Resources

- `serial://ports` - Available ports list
- `serial://port/{port_name}` - Port details

## Install

```bash
pip install mcp-rs232
```

---
> Source: [daedalus/mcp-rs232](https://github.com/daedalus/mcp-rs232) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

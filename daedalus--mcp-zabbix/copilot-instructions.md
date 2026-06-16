## mcp-zabbix

> MCP server for Zabbix API - exposes all Zabbix API functionality.

# MCP Zabbix

MCP server for Zabbix API - exposes all Zabbix API functionality.

## When to use this skill

Use this skill when you need to:
- Monitor infrastructure via Zabbix
- Query hosts and items
- Manage triggers
- Get monitoring data

## Tools

Exposes all Zabbix API methods as tools following pattern:
- `zabbix_host_get`, `zabbix_host_create`, etc.
- `zabbix_item_get`, `zabbix_item_create`, etc.
- `zabbix_trigger_get`, etc.

## Configuration

- `ZABBIX_URL` - Zabbix API URL
- `ZABBIX_TOKEN` - API token (or user/password)

## Install

```bash
pip install mcp-zabbix
```

---
> Source: [daedalus/mcp-zabbix](https://github.com/daedalus/mcp-zabbix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

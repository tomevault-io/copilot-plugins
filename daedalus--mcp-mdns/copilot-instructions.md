## mcp-mdns

> MCP server for mDNS (Multicast DNS) service discovery.

# MCP mDNS

MCP server for mDNS (Multicast DNS) service discovery.

## When to use this skill

Use this skill when you need to:
- Discover services on local network
- Find .local hostnames
- Query network services (printers, servers, etc.)
- Register/unregister services

## Tools

- `mdns_list_service_types` - List all advertised service types
- `mdns_browse_services` - Browse services by type
- `mdns_get_service_info` - Get service details (host, port, TXT)
- `mdns_resolve_hostname` - Resolve .local hostname to IP
- `mdns_register_service` - Register new service
- `mdns_unregister_service` - Unregister service

## Install

```bash
pip install mcp-mdns
```

---
> Source: [daedalus/mcp-mdns](https://github.com/daedalus/mcp-mdns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

## integrations

> Integration client patterns


# Integration Clients

## Directory Structure

```
app/integrations/
├── clients/
│   ├── grafana/          # Vendor-specific client packages
│   ├── datadog/
│   ├── s3_client.py      # Single-file clients
│   └── vercel.py
├── opensre_mcp.py        # MCP server implementation
└── ...
```

## Adding a New Integration

1. Create a client under `app/integrations/clients/<vendor>/`
2. Implement connection, authentication, and API methods
3. Create a tool under `app/tools/<VendorTool>/` that uses the client
4. Wire availability into `resolve_integrations` node if the integration requires runtime discovery
5. Add the vendor to `EvidenceSource` literal in `app/state.py` if it's a new source type

## Conventions

- Clients should be stateless or use context managers for connection lifecycle
- Return `{"success": bool, "data": ..., "error": ...}` from client methods
- Keep API-specific logic in the client; tool handles param extraction and result formatting
- MCP tools are registered via `opensre-mcp` entry point in `pyproject.toml`

---
> Source: [Tracer-Cloud/opensre](https://github.com/Tracer-Cloud/opensre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->

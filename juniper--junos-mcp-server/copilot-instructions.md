## junos-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server for Juniper Junos network devices. It enables LLMs to interact with Juniper network equipment through a standardized interface using the FastMCP framework and Juniper's PyEZ library.

## Key Commands

### Running the Server

```bash
# Install dependencies
pip install -r requirements.txt

# Run with stdio transport (for Claude Desktop)
python3.11 jmcp.py -f devices.json -t stdio

# Run with streamable-http transport (for VSCode)
python3.11 jmcp.py -f devices.json -t streamable-http -H 127.0.0.1 -p 30030

# Docker build
docker build -t junos-mcp-server:latest .

# Docker run (stdio)
docker run --rm -it -v /path/to/devices.json:/app/config/devices.json junos-mcp-server:latest

# Docker run (streamable-http)
docker run --rm -it -v /path/to/devices.json:/app/config/devices.json -p 30030:30030 junos-mcp-server:latest python jmcp.py -f /app/config/devices.json -t streamable-http -H 0.0.0.0
```

### Testing and Development

- No automated tests are currently configured
- No linting configuration is present
- Manual testing requires a valid Junos device configuration in JSON format

## Architecture

The server implements six MCP tools in `jmcp.py`:

1. **execute_junos_command** - Execute arbitrary CLI commands on routers
2. **get_junos_config** - Retrieve device configuration (uses `show configuration | display inheritance no-comments`)
3. **junos_config_diff** - Compare configuration versions (rollback comparison)
4. **gather_device_facts** - Collect device information using PyEZ facts
5. **get_router_list** - List available routers from the configuration
6. **load_and_commit_config** - Apply configuration changes (supports set/text/xml formats)

### Key Implementation Details

- All device connections use the `_run_junos_cli_command` helper function (jmcp.py:81)
- Connection parameters are prepared by `prepare_connection_params` (jmcp.py:42) which handles both password and SSH key authentication
- Default timeout is 360 seconds for long-running operations
- The server uses FastMCP for the MCP protocol implementation
- Device configurations are loaded from a JSON file at startup

### Device Configuration Format

The device configuration file must follow this structure:

```json
{
    "router-name": {
        "ip": "ip-address",
        "port": 22,
        "username": "user",
        "auth": {
            "type": "password|ssh_key",
            "password": "pwd",  // if type is password
            "private_key_path": "/path/to/key.pem"  // if type is ssh_key
        }
    }
}
```

## Security Considerations

- **CRITICAL**: Configuration changes are automatically committed to devices when using `load_and_commit_config`
- SSH key authentication is strongly recommended over passwords
- The server exposes network infrastructure to LLM access - ensure corporate policies allow this
- Always review LLM-generated configurations before allowing execution

## Common Development Tasks

When modifying the server:

1. **Adding new tools**: Follow the existing pattern using `@mcp.tool()` decorator
2. **Error handling**: Use try/except blocks around device operations to catch ConnectError and general exceptions
3. **Authentication**: Any changes must support both password and SSH key authentication types
4. **Logging**: Use the global `log` logger for debugging (`logging.getLogger('jmcp-server')`)

## Integration Points

- **Claude Desktop**: Use stdio transport with absolute paths in configuration
- **VSCode with GitHub Copilot**: Use streamable-http transport pointing to server URL
- **Docker**: Mount device configuration and any SSH key files as volumes

---
> Source: [Juniper/junos-mcp-server](https://github.com/Juniper/junos-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->

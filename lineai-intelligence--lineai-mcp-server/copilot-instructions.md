## error-handling

> Error handling patterns for the Lineai MCP Server

# Use the following pattern for error handling in tool implementations

```python
try:
    # Operations that might fail
except Exception as e:
    sys.stderr.write(f"Error: {str(e)}\n")
    return [types.TextContent(type="text", text=f"# Error\n\n{str(e)}")]
```

- Always catch and report exceptions
- Write errors to stderr
- Return formatted error messages to the client

---
> Source: [lineai-intelligence/lineai-mcp-server](https://github.com/lineai-intelligence/lineai-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->

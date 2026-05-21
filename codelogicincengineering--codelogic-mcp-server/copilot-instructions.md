## error-handling

> Error handling patterns for the CodeLogic MCP Server

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
> Source: [CodeLogicIncEngineering/codelogic-mcp-server](https://github.com/CodeLogicIncEngineering/codelogic-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->

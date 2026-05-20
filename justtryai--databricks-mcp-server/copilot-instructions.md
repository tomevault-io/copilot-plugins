## mcp-tools

> MCP Tool Implementation Guidelines

# MCP Tool Implementation Standards

## Function Signature
- Tool functions should be async
- Follow the pattern:
  ```python
  async def tool_name(params: Dict[str, Any]) -> Dict[str, Any]:
      """Tool description.
      
      Args:
          params: Dictionary of parameters from MCP client
          
      Returns:
          Dictionary adhering to MCP protocol response format
      """
  ```

## Documentation Requirements
Each tool must have documentation with:
- name
- description
- parameters
- returns

Example:
```python
"""
name: list_clusters
description: Lists all available Databricks clusters
parameters: {}
returns: List of cluster objects
"""
```

## Error Handling
- All tool functions must return errors as part of the result object with `isError: true`
- Example:
  ```python
  {
      "result": None,
      "isError": True,
      "errorMessage": "Unable to connect to Databricks API"
  }
  ```

## Performance Considerations
- Implement appropriate timeouts for all external calls
- Long-running operations should provide progress updates
- Handle Databricks API rate limiting and retries

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->

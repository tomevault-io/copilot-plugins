## databricks-api

> Databricks API Client Guidelines

# Databricks API Implementation Standards

## Function Design
- API functions should be async
- Follow the pattern:
  ```python
  async def api_call(client, **params):
      """Short description of the API call.
      
      Args:
          client: Databricks client instance
          **params: API-specific parameters
          
      Returns:
          Processed and validated API response
          
      Raises:
          APIError: When the API call fails
      """
  ```

## Error Handling
- Provide clear error messages with context about the failure
- Implement retry logic for transient failures
- Handle rate limiting gracefully
- Example:
  ```python
  try:
      response = await client.make_request(...)
  except RateLimitException:
      await asyncio.sleep(retry_after)
      response = await client.make_request(...)
  except APIException as e:
      raise APIError(f"Failed to call {api_name}: {str(e)}")
  ```

## Response Validation
- Validate responses before returning to ensure they match expected schema
- Return properly typed objects, not just raw JSON

## Performance
- Implement appropriate timeouts
- Use bulk operations where possible
- Cache responses when appropriate

## References
- Databricks API: https://docs.databricks.com/api/azure/workspace/clusters/edit

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->

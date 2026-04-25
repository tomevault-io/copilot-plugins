## adventure-agent

> All graph nodes should handle errors gracefully:

# Error Handling Patterns

## Node Error Handling
All graph nodes should handle errors gracefully:

```python
async def agent_node(state: AdventureState) -> Dict[str, Any]:
    """Agent node with error handling."""
    try:
        # Agent logic
        result = await agent.method(...)
        return {
            "output": result,
            "completed_agents": state.completed_agents + ["agent_name"],
        }
    except Exception as e:
        error_msg = f"Agent name error: {str(e)}"
        return {
            "output": None,  # or empty list/dict
            "completed_agents": state.completed_agents + ["agent_name"],
            "errors": state.get("errors", []) + [error_msg],
        }
```

## Error Accumulation
- Accumulate errors in `state.errors` list
- Don't raise exceptions from nodes (breaks graph execution)
- Return partial results when possible
- Log errors for debugging

## Tool Error Handling
Tools should return empty results on errors:

```python
@tool
def tool_name(param: str) -> str:
    """Tool that handles errors gracefully."""
    try:
        # Tool logic
        return json.dumps(result)
    except Exception as e:
        print(f"Tool error: {e}")
        return json.dumps({"error": str(e), "results": []})
```

## Retry Policies
Use retry policies for external API calls:

```python
from langgraph.types import RetryPolicy

api_retry_policy = RetryPolicy(
    max_attempts=3,
    initial_interval=1.0,
    backoff_factor=2.0,
)

graph.add_node("agent_node", agent_node_function, retry_policy=api_retry_policy)
```

## Fallback Values
Always provide fallback values:
- Empty lists `[]` for list fields
- `None` or empty dict `{}` for optional fields
- Default strings `""` for string fields
- Default activity type: `"mountain_biking"`

## Error Messages
- Use descriptive error messages: `f"Agent name error: {str(e)}"`
- Include context in error messages
- Don't expose sensitive information in errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blakebauman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

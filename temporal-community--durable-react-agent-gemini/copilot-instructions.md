## durable-react-agent-gemini

> This is a Temporal-based agentic loop implementation using Google's Gemini model. It demonstrates how to build durable AI agents with tool-calling capabilities. The implementation is consolidated into a single worker file for educational clarity.

# CLAUDE.md - Gemini Agentic Loop Project

## Project Overview

This is a Temporal-based agentic loop implementation using Google's Gemini model. It demonstrates how to build durable AI agents with tool-calling capabilities. The implementation is consolidated into a single worker file for educational clarity.

## Project Structure

```
agentic_loop_tool_call_gemini_python/
├── durable_agent_worker.py # All-in-one: workflow, activities, tools, and worker
├── start_workflow.py       # Client (no API key needed)
├── pyproject.toml          # Dependencies (pip install .)
└── .env                    # GOOGLE_API_KEY (not checked in)
```

## Agent Architecture

The agent is composed of two loosely coupled parts, both defined in `durable_agent_worker.py`:

1. **Agent Engine** (the `AgentWorkflow` class): The agentic loop that orchestrates LLM calls and tool execution. This is generic and doesn't change when adding/removing tools.

2. **Agent Definition**: What makes this agent unique:
   - `SYSTEM_INSTRUCTIONS`: System instructions defining personality and behavior
   - Tool functions and registry (`get_handler`, `get_tools`): Available capabilities the agent can use

This separation (within the single file) allows easy customization by modifying the definition sections without touching the engine.

## Key Implementation Details

### Tool Definition Strategy

We use `FunctionDeclaration.from_callable_with_api_option()` from the Google GenAI SDK to generate tool definitions. Unlike `from_callable()` (which requires a `genai.Client`), this method accepts the API backend as a string (`"GEMINI_API"`), so no client or API key is needed for tool generation.

**Docstrings for parameter descriptions**: The SDK extracts the entire docstring as the function description, but does NOT extract individual parameter descriptions from Pydantic `Field(description=...)`. Put parameter descriptions in the docstring's Args section.

### Why Pydantic Models for Tool Parameters

Temporal best practice is to use 0-1 parameters for activities, typically a data structure. Benefits:
- Backward compatibility when adding optional fields
- Clear activity contracts
- Consistent patterns across the codebase

When using Pydantic models with `from_callable()`, the LLM produces **nested output**:
```python
{"request": {"state": "CA"}}  # Not flat {"state": "CA"}
```

The `dynamic_tool_activity` extracts the nested dict using the parameter name.

### Temporal Sandbox Considerations

- The `workflow.unsafe.imports_passed_through()` block at module level imports `google.genai`, `pydantic_core`, `annotated_types`, `httpx`, and `pydantic` to avoid sandbox restrictions. `httpx` in particular is restricted because it internally subclasses `urllib.request.Request`.
- Activities execute outside the sandbox at runtime, so their use of `genai.Client` in function bodies is fine

### Serialization

`types.Tool`, `types.Content`, and `types.Part` from Google GenAI SDK are Pydantic models. Temporal's `pydantic_data_converter` handles serialization automatically - no manual serialization/deserialization code is needed.

The workflow uses native SDK types directly:
```python
contents: list[types.Content] = [
    types.Content(role="user", parts=[types.Part(text=input)])
]
```

### Gemini SDK Retry and Auto-Call Disabled

The Gemini SDK has two built-in behaviors that conflict with Temporal's durability model:

1. **Automatic function calling**: The SDK can silently execute tool calls client-side. Disabled via `AutomaticFunctionCallingConfig(disable=True)` so that tool calls are tracked in workflow history and can be replayed on recovery.

2. **Built-in retries**: The SDK retries failed requests (5 attempts with exponential backoff by default). Disabled via `HttpRetryOptions(attempts=1)` so that Temporal owns all retry behavior. Leaving SDK retries on can interfere with Temporal's timeout tracking and durable error handling.

Both are disabled when constructing the `genai.Client` in the `generate_content` activity.

### String-Based Workflow Execution

The client (`start_workflow.py`) uses string-based workflow execution to avoid importing the worker module:
```python
result = await client.execute_workflow(
    "AgentWorkflow",  # String, not AgentWorkflow.run
    query,
    ...
)
```

This keeps the client script simple and avoids importing the worker module.

## Common Pitfalls

1. **Forgetting docstring parameter descriptions**: The LLM won't know what parameters mean
2. **Not handling nested LLM output**: Tool invocation will fail
3. **Wrong type for function_calls args**: Use `dict[str, Any]` not `dict[str, str]` since args is a dict
4. **Pydantic sandbox warnings**: Import `pydantic_core` and `annotated_types` early in `imports_passed_through`

## Customizing the Agent

### Changing Agent Behavior

Edit the `SYSTEM_INSTRUCTIONS` constant in `durable_agent_worker.py` to modify the system instructions that define the agent's personality and behavioral guidelines.

### Adding New Tools

1. Create Pydantic model for parameters in `durable_agent_worker.py`
2. Create async handler with descriptive docstring (include parameter descriptions!)
3. Register in the same file:
   - Add to `get_handler()`
   - Add `FunctionDeclaration.from_callable_with_api_option()` in `get_tools()`

No changes to the workflow are needed when adding tools.

## Environment Variables

- `GOOGLE_API_KEY`: Required by worker only (for Gemini API calls)

## Testing

Install dependencies:
```bash
pip install .
```

Run the worker:
```bash
python -m durable_agent_worker
```

Run a query (no API key needed):
```bash
python -m start_workflow "What's the weather in California?"
```

---
> Source: [temporal-community/durable-react-agent-gemini](https://github.com/temporal-community/durable-react-agent-gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->

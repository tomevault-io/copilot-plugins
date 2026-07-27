## gofannon

> **Critical**: All LLM calls in this codebase MUST go through the centralized `call_llm` function from `services/llm_service.py`. Do NOT use `litellm` directly.

# Agent Guidelines for the Gofannon User Service

## LLM Calls: Use `call_llm`, NOT `litellm` Directly

**Critical**: All LLM calls in this codebase MUST go through the centralized `call_llm` function from `services/llm_service.py`. Do NOT use `litellm` directly.

### Why This Matters

The `call_llm` function provides essential functionality that direct `litellm` calls bypass:

1. **Cost Tracking**: Automatically tracks API costs and associates them with users
2. **User Allowance Management**: Enforces spending limits and prevents unauthorized usage
3. **Observability**: Logs all LLM calls for debugging and monitoring
4. **Consistent Error Handling**: Provides standardized error logging and propagation
5. **Provider Configuration**: Handles provider-specific configurations (API styles, response formats)
6. **Reasoning Content Extraction**: Properly extracts thoughts/reasoning from extended thinking models

### Correct Usage

```python
from services.llm_service import call_llm, stream_llm

# For standard completions
content, thoughts = await call_llm(
    provider="openai",
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello"}],
    parameters={"temperature": 0.7},
    user_service=user_service,  # Pass None if not tracking user costs
    user_id=user_id,            # Pass None if not tracking user costs
)

# For streaming responses
async for chunk in stream_llm(
    provider="openai",
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello"}],
    parameters={"temperature": 0.7},
    user_service=None,
    user_id=None,
):
    # Process chunk
    pass
```

### Incorrect Usage (DO NOT DO THIS)

```python
# WRONG - Do not import litellm directly
import litellm
response = await litellm.acompletion(model="openai/gpt-4", messages=[...])

# WRONG - Do not use litellm.acompletion
from litellm import acompletion
response = await acompletion(model="openai/gpt-4", messages=[...])
```

### Agent Code Execution

When writing code for Gofannon agents (code that runs in the agent sandbox), use `call_llm` which is available in the execution context:

```python
async def run(input_dict, tools):
    # call_llm is available in the sandbox
    content, thoughts = await call_llm(
        provider="openai",
        model="gpt-4",
        messages=[{"role": "user", "content": input_dict["query"]}],
        parameters={},
        user_service=None,
        user_id=None,
    )
    return {"outputText": content}
```

## Files That Should ONLY Import `litellm`

The only files that should directly import `litellm` are:

1. `services/llm_service.py` - The centralized LLM service (this is the abstraction layer)
2. `services/litellm_logger.py` - The logging integration for litellm

All other files should import from `llm_service`:

```python
from services.llm_service import call_llm, stream_llm
```

## Testing

When writing tests, mock `call_llm` and `stream_llm` rather than `litellm`:

```python
from unittest.mock import AsyncMock, patch

# Correct - mock call_llm
with patch("your_module.call_llm", new_callable=AsyncMock) as mock:
    mock.return_value = ("response content", None)
    # ... test code

# Correct - mock stream_llm
async def mock_stream(*args, **kwargs):
    yield MockChunk("Hello")
    yield MockChunk(" World")

with patch("your_module.stream_llm", side_effect=mock_stream):
    # ... test code
```

---
> Source: [The-AI-Alliance/gofannon](https://github.com/The-AI-Alliance/gofannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->

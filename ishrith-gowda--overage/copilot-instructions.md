## overage

> Testing patterns and conventions for Overage


# Testing Patterns

## Test Naming

`test_<function>_<scenario>_<expected_result>`

```python
async def test_extract_reasoning_tokens_valid_response_returns_correct_count(self): ...
async def test_forward_request_timeout_raises_timeout_error(self): ...
async def test_list_calls_no_auth_returns_401(self): ...
```

## Fixtures

Use fixtures from `proxy/tests/conftest.py`:
- `client` — httpx.AsyncClient bound to test app
- `db_session` — isolated in-memory SQLite session
- `test_api_key` — valid API key string
- `mock_openai_response` — realistic OpenAI response factory
- `mock_anthropic_response` — realistic Anthropic response factory
- `sample_call_log` — pre-built APICallLog record
- `sample_estimation` — pre-built EstimationResult record

## Mocking External Services

```python
from unittest.mock import AsyncMock, MagicMock

mock_response = MagicMock(spec=httpx.Response)
mock_response.status_code = 200
mock_response.json.return_value = mock_openai_response(reasoning_tokens=5000)
provider._client.post = AsyncMock(return_value=mock_response)
```

## Every Test Must Assert

```python
# WRONG — no assertion
async def test_something(self, client):
    await client.get("/health")

# CORRECT
async def test_health_returns_200(self, client):
    response = await client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] in ("healthy", "degraded")
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ishrith-gowda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

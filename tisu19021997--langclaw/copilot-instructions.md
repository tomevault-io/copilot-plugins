## testing

> Testing conventions for langclaw


# Testing

## Setup

- Framework: pytest + pytest-asyncio (`asyncio_mode = "auto"` — no need for `@pytest.mark.asyncio`)
- Run: `uv run pytest tests/ -v`

## Patterns

- Use `monkeypatch.setenv()` for env var overrides — never mutate `os.environ` directly
- Use factory functions (`make_message_bus`, `make_checkpointer_backend`) for integration tests
- Class-based grouping for related tests (e.g. `TestSplitMessage`, `TestIsAllowed`)
- Prefer real implementations over mocks when the component is fast and deterministic (buses, config)
- For async tests, just define `async def test_...` — the auto mode handles the rest

## Assertions

- Assert tool error responses with `assert result == {"error": ...}`
- For bus tests, use `async with bus:` and break after first consumed message
- For config tests, use `monkeypatch.setenv("LANGCLAW__...", value)` then call `load_config()`

---
> Source: [tisu19021997/langclaw](https://github.com/tisu19021997/langclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->

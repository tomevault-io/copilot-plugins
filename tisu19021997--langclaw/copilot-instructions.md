## extending-the-framework

> When adding new tools, channels, middleware, message bus backends, or checkpointer backends to the langclaw framework


# Extending Langclaw

## Adding a Tool

Tools are async functions decorated with `@tool` from `langchain_core.tools` or registered via `@app.tool()` on the `Langclaw` instance. Follow the pattern in `langclaw/agents/tools/`.

- Return a dict on failure: `{"error": f"Failed to ...: {exc}"}` — never raise
- Add type hints to all parameters — LangChain uses them for the tool schema
- The docstring becomes the tool description the LLM sees — make it actionable
- Register in `langclaw/agents/tools/__init__.py` if it's a built-in tool

## Adding a Channel

Extend `BaseChannel` in `langclaw/gateway/base.py`. See `telegram.py`, `discord.py`, `websocket.py` for reference.

- Implement `start()`, `stop()`, `send()`, and `send_ai_message()`
- The channel publishes `InboundMessage` to the bus — it never talks to the agent directly
- Set `origin="channel"` when publishing user messages to the bus
- Add the optional dependency to `pyproject.toml` under `[project.optional-dependencies]`
- Guard the import in `Langclaw._build_all_channels()` with `try/except ImportError`

## Adding a Bus or Checkpointer Backend

Follow the abstract-base + factory pattern:

1. Create the implementation in the respective package (e.g. `bus/redis_bus.py`)
2. Extend the abstract base (`bus/base.py` or `checkpointer/base.py`)
3. Implement the async context manager protocol (`__aenter__` / `__aexit__`)
4. Register in the factory function (`make_message_bus` or `make_checkpointer_backend`)
5. Add config options to `config/schema.py` following the existing nested pattern

## Adding Middleware

Middleware uses the `@wrap_model_call` pattern from deepagents. See `middleware/permissions.py` for the canonical example.

- Middleware filters or transforms the tool list **before** the LLM sees it
- Order matters — middleware is composed in `agents/builder.py`
- Accept `LangclawConfig` in the factory, return the middleware callable

## Config Changes

When adding config, update both:
1. `langclaw/config/schema.py` — add the Pydantic model/field
2. `.env.example` — document the new env var with the `LANGCLAW__` prefix

---
> Source: [tisu19021997/langclaw](https://github.com/tisu19021997/langclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->

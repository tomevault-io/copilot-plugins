## wechat-gptbot

> - **Use this document as the source of truth** for the project's architecture, conventions, and constraints. Follow its guidelines throughout the conversation.

### Project Awareness & Context

- **Use this document as the source of truth** for the project's architecture, conventions, and constraints. Follow its guidelines throughout the conversation.
- **Check `TASK.md`** before starting a new task. If the task isn't listed, add it with a brief description and today's date.

### Project Overview

wechat-gptbot is a WeChat bot powered by ChatGPT and other LLMs, running via Windows DLL hook injection. It supports multi-turn conversations, image generation (DALL-E-3), and an extensible plugin system.

### AI Behavior Rules

- **Never assume missing context. Ask questions if uncertain.**
- **Never hallucinate libraries or functions** – only use known, verified Python packages.
- **Always confirm file paths and module names** exist before referencing them in code or tests.
- **Never delete or overwrite existing code** unless explicitly instructed to or if part of a task from `TASK.md`.

### Task Completion

- **Mark completed tasks in `TASK.md`** immediately after finishing them.
- Add new sub-tasks or TODOs discovered during development to `TASK.md` under a "Discovered During Work" section.

### Documentation & Explainability

- **Keep documentation in sync with code changes**:
  - `CLAUDE.md` — update when project structure, conventions, or architecture changes
  - `README.md` / `README_ZH.md` — update when adding features, changing commands, or modifying setup steps
  - `plugins/README.md` — update when changing the plugin API
- **Comment non-obvious code** and ensure everything is understandable to a mid-level developer.
- When writing complex logic, **add an inline `# Reason:` comment** explaining the why, not just the what.

### Testing & Reliability

- **Always create Pytest unit tests for new features** (functions, classes, routes, etc).
- **After updating any logic**, check whether existing unit tests need to be updated. If so, do it.
- **Tests should live in a `/tests` folder** mirroring the main app structure.
  - Include at least:
    - 1 test for expected use
    - 1 edge case
    - 1 failure case

#### Key Entry Points

- **`app.py`**: Main entry point — loads config, initializes plugins, starts WeChat channel
- **`config.py`**: Configuration loader — reads `config.json` into a global dict

#### Module Layout

- **`bot/`**: LLM implementations
  - `bot.py` — Factory pattern: selects ChatGPT, LiteLLM, or Azure based on config
  - `chatgpt.py` — OpenAI ChatGPT implementation
  - `litellm.py` — LiteLLM multi-model support (Claude, Cohere, etc.)
  - `azure_chatgpt.py` — Azure OpenAI implementation
- **`channel/`**: WeChat channel and messaging
  - `channel.py` — Abstract channel interface
  - `message.py` — Message model and parsing
  - `wechat.py` — WebSocket handler for WeChat DLL hook communication
- **`common/`**: Core utilities and patterns
  - `singleton.py` — `@singleton` decorator
  - `emitter.py` — Event emitter/observer pattern for plugin system
  - `context.py` — Context model for chat conversations
  - `reply.py` — Reply types and model
  - `session.py` — Session and conversation memory with TTL expiry
  - `expired_dict.py` — Time-based expiring dictionary
- **`plugins/`**: Plugin system
  - `plugin.py` — Abstract plugin base class
  - `event.py` — `EventType` and `Event` definitions (DID_RECEIVE_MESSAGE, WILL_GENERATE_REPLY, WILL_DECORATE_REPLY, WILL_SEND_REPLY)
  - `manager.py` — Plugin loader and lifecycle manager
  - `built_in.py` — Built-in commands plugin (#clear session, #query key, #help)
  - `source.json` — External plugin registry
- **`utils/`**: Utility functions
  - `api.py` — WeChat API calls (HTTP/WebSocket)
  - `const.py` — Constants and message types
  - `log.py` — Logging setup
  - `serialize.py` — File/message serialization
  - `check.py` — Validation helpers
  - `query_key.py` — OpenAI API key usage query
  - `package.py` — Package management for plugins
  - `gen.py` — ID generation

### Code Structure & Modularity

- **Never create a file longer than 500 lines.** If a file approaches this limit, refactor by splitting it into modules.
- **Organize code by responsibility**: `bot/` for LLM logic, `channel/` for messaging transport, `common/` for shared models, `plugins/` for extensibility, `utils/` for helpers.
- **Respect the established design patterns**:
  - **Singleton** (`@singleton` decorator): `Bot`, `WeChatChannel`, `PluginManager` — do not create additional instances
  - **Observer/Emitter**: Plugin events flow through `Emitter` — use `EventType` enum for new events
  - **Factory**: `Bot` class selects the LLM implementation based on config — extend here for new providers
  - **Abstract Base Class**: `Channel` and `Plugin` define interfaces — new implementations must inherit from these

#### Message Flow

1. WebSocket receives raw message from WeChat DLL hook
2. Message parsed into `Message` object
3. Plugins emit `DID_RECEIVE_MESSAGE` event
4. `Context` created based on chat type (single/group)
5. Plugins emit `WILL_GENERATE_REPLY` event
6. `Bot` generates reply via selected LLM
7. Plugins emit `WILL_DECORATE_REPLY` event
8. Reply decorated with prefix/suffix
9. Plugins emit `WILL_SEND_REPLY` event
10. Reply sent via WebSocket

### Style & Conventions

#### Python Code Standards

- **Python target version**: 3.13
- **Formatter**: Black (line length 88)
- **Linter**: Ruff (pycodestyle `E` + Pyflakes `F`, ignoring `E501`)
- **Max cyclomatic complexity**: 10
- **Naming conventions**:
  - `snake_case` for functions, methods, variables, and module file names
  - `PascalCase` for classes and enums
  - `UPPER_SNAKE_CASE` for constants
  - Prefix private helpers with `_`
- **Type hints**: Use type annotations for function signatures. The project uses Pydantic (`BaseModel`) for data models — follow this pattern for new models.
- **Imports**: Standard library first, then third-party, then project modules. One import per line for project modules.
- **Configuration**: All runtime config lives in `config.json` (loaded via `config.py`). Access values through `conf()` — never hardcode configurable values.
- **Error handling**: Use `logger.error()` / `logger.exception()` for error reporting. Raise descriptive exceptions for startup failures.
- **Logging**: Use `from utils.log import logger` — do not use `print()` for runtime output.
- Write **docstrings for every function** using the Google style:

  ```python
  def example():
      """
      Brief summary.

      Args:
          param1 (type): Description.

      Returns:
          type: Description.
      """
  ```

#### Plugin Development

- Extend the `Plugin` base class with the `@register` decorator
- Implement event hooks: `did_receive_message`, `will_generate_reply`, `will_decorate_reply`, `will_send_reply`
- Control flow with `event.proceed()`, `event.stop()`, or `event.bypass()`
- See `plugins/README.md` for the full plugin development guide

### Build & Run

```bash
# Install dependencies
pip install -r requirements.txt

# Set up pre-commit hooks
pre-commit install

# Create config from template
cp config.template.json config.json
# Edit config.json with your API key and settings

# Run the bot
python app.py
```

**Requirements**: Python >= 3.13, Windows with WeChat 3.2.1.121-LTS + wechat-dll-injector

### Linting & Formatting

```bash
# Format code
black .

# Lint code
ruff check .

# Lint and auto-fix
ruff check --fix .
```

Pre-commit hooks run Black and Ruff automatically on staged files. CI runs Ruff via GitHub Actions on push and pull requests.

---
> Source: [iuiaoin/wechat-gptbot](https://github.com/iuiaoin/wechat-gptbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

## aeloon

> - Install for development: `pip install -e ".[dev]"`

# Repository Guidelines

## Common Commands

- Install for development: `pip install -e ".[dev]"`
- Run the CLI locally: `aeloon --help`
- First-time setup / config bootstrap: `aeloon onboard`
- One-shot agent run: `aeloon agent -m "Hello!"`
- Start gateway mode: `aeloon gateway`
- Inspect configured channels: `aeloon channels status`
- Run tests: `pytest`
- Run one test file: `pytest tests/test_kernel.py`
- Run matching tests: `pytest -k "wechat"`
- Lint and auto-fix: `ruff check --fix .`
- Format: `ruff format .`

Use `ruff==0.15.6`; CI expects that exact version.

## Architecture Overview

- Aeloon is a message-bus-based assistant runtime. The main flow is: channel receives inbound event → `MessageBus` → `Dispatcher` → `AgentLoop` → provider/tool execution → outbound message routed back through `ChannelManager`.
- `aeloon/agent/loop.py` is the orchestration layer around the reusable kernel. It wires together context building, tool registration, sessions, memory consolidation, skill runtime, subagents, cron context, optional MCP connections, and outbound publishing.
- `aeloon/agent/kernel.py` and `aeloon/agent/task_graph.py` handle one LLM turn. Tool-call batches are compiled into an internal DAG so independent read-only work can run concurrently while mutating/exclusive operations are serialized conservatively.
- `aeloon/agent/dispatcher.py` sits in front of the loop and owns slash-command handling, per-session task lifecycle, cancellation, and WeChat login/status coordination.
- `aeloon/channels/` is one integration per file, all inheriting `BaseChannel`. Channel discovery is automatic via registry scanning plus entry-point plugins; do not add manual registration for built-in channels.
- `aeloon/channels/manager.py` owns channel startup/shutdown, lifecycle state tracking (`pending`, `starting`, `running`, `failed`, `stopped`), and outbound routing from the bus back to each channel.
- `aeloon/providers/registry.py` is the source of truth for provider metadata and matching. Provider selection is driven by explicit provider name, model keywords/prefixes, API key prefixes, and API base URLs.
- `aeloon/config/schema.py` + `aeloon/config/loader.py` define the Pydantic config model and loading precedence. Config accepts camelCase aliases, and the effective file path is `--config` flag → `AELOON_CONFIG` → `~/.aeloon/config.json`.
- `aeloon/skills/` are markdown-driven capabilities. Skills are injected into the prompt progressively; workspace skills override built-ins. Helper scripts live alongside a skill and are typically invoked through tool execution.
- `aeloon/templates/AGENTS.md` is the runtime prompt template for end-user agent behavior; it is separate from this repository-level `AGENTS.md`.
- `aeloon/channels/wechat.py` and `aeloon/channels/wechat_ilink/` implement the native WeChat login/runtime flow. `/wechat login|logout|status` behavior is coordinated through the dispatcher and channel manager.
- `bridge/` is a separate Node.js/TypeScript WhatsApp bridge process. `bridge/src/server.ts` binds WebSocket only to `127.0.0.1` and optionally enforces a bridge token; treat it as local-only infrastructure.

## Codebase-Specific Rules

- Follow `CLAUDE.md` across the repo: keep CLI code thin in `aeloon/cli/`; put real logic in the runtime/core modules instead of command handlers.
- Public functions and class methods require type hints.
- Prefer `async`/`await` for I/O paths; keep sync wrappers thin.
- Do not import channel-specific dependencies at module top level; use lazy imports so optional integrations stay optional.
- Tool implementations should return `"Error: ..."` strings instead of swallowing exceptions so the model can recover on the next iteration.
- Do not hardcode user data/config paths; use config/path helpers such as `Config.workspace_path` and helpers in `aeloon/config/paths.py`.
- Do not rename camelCase arguments that intentionally mirror external APIs; use `# noqa: N803` when needed.

## Testing Notes

- The pytest suite uses `asyncio_mode = "auto"`, so async tests do not need explicit decorators beyond normal pytest async patterns already used nearby.
- Existing tests are organized close to module boundaries, typically as `tests/test_<module>.py`; follow that pattern when adding coverage.
- Mock external services and provider/network interactions; avoid real API calls in tests.

---
> Source: [AetherHeart-AI/Aeloon](https://github.com/AetherHeart-AI/Aeloon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->

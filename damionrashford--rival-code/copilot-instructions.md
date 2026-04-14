## rival-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run the CLI (dev, from repo root)
python cli.py "your task"
python cli.py chat

# Run installed entry point (after uv sync)
uv run rival "your task"
uv run rival chat

# Build package
uv build

# Install with all optional deps
uv pip install -e ".[all]"

# Lint
uv run ruff check .
uv run ruff check --fix .

# Type check
uv run mypy rival_code/ cli.py

# Run a single test
uv run pytest tests/test_foo.py::test_name -v
```

## Architecture

The CLI entry point is `rival_code/cli.py` (the `main()` function handles the bare `rival "task"` shorthand by inserting `run` into argv). `cli.py` at repo root is a thin shim for local dev.

**Request flow:**
1. `cli.py main()` → Click command → `_setup()` assembles `Config`
2. `build_agent(cfg)` wires all tools into an `Agent`
3. `run_agent(agent, task, session=sess)` streams the agentic loop via `Runner.run_streamed()`
4. Approval interruptions are caught mid-stream, prompted inline, then the state is resumed

**`build_agent()` assembly order** (`agent.py`):
- Provider client (`AsyncOpenAI(api_key=cfg.api_key(), base_url=cfg.base_url)`) → `OpenAIResponsesModel` (Responses API, required for `ComputerTool`)
- `load_settings()` → `get_permissions()` → tool filter deny-list
- Skills discovery → `activate_skill` tool + system prompt catalog
- MCP servers discovery → system prompt catalog (no SDK protocol — agent calls `python cli.py call-tool` via `shell`)
- Sub-agent specs discovery → each becomes a `@function_tool`; `persistent: true` specs wrap `Runner.run()` with a dedicated `JSONLSession`
- `run_parallel` meta-tool (only when sub-agents exist) — `asyncio.gather` over sub-agent tools
- `ShellTool(executor=ShellExecutor(...))` + `ApplyPatchTool(editor=LocalEditor(...))`
- `ComputerTool` (optional, `--computer` flag, wraps `ChromeDevToolsComputer`)
- `filter_tools(all_tools, permissions)` removes deny-listed tool names

## Key Modules

| Module | Role |
|--------|------|
| `agent.py` | `build_agent()` factory + `run_agent()` streaming loop |
| `config.py` | `Config` dataclass; all constants live here |
| `prompts.py` | `make_system_prompt()` — returns async callable injecting live `git status` (30s cache) + RIVAL.md content |
| `tools.py` | Read-only function tools (`read_file`, `list_directory`, `search_in_files`, `get_file_tree`, `web_search`, `web_fetch`); web search has 5-min module-level cache |
| `editor.py` | `LocalEditor` — applies unified diffs; `dry_run=True` returns diff preview instead of writing |
| `executor.py` | `ShellExecutor` — runs shell with configurable timeout; `needs_approval_fn` marks dangerous commands for interrupt |
| `hooks.py` | `RivalCodeHooks` — displays tool events, mirrors output to `--output` file |
| `session.py` | `JSONLSession` — implements `SessionABC`; one JSONL file per session at `~/.rival/sessions/` |
| `costs.py` | Appends token usage records to `~/.rival/costs/<session_id>.jsonl` after each run |
| `settings.py` | Merges `~/.rival/settings.json` + `{cwd}/.rival/settings.json`; `filter_tools()` removes denied tools |
| `skills.py` | Implements agentskills.io spec; 3-tier disclosure (catalog → full body → resources); scans `.agents/skills/`, `.claude/skills/`, `.rival/skills/` |
| `subagents.py` | Discovers `.rival/agents/*.md` + `~/.rival/agents/*.md`; persistent agents share a `JSONLSession` |
| `mcp.py` | MCP-via-CLI: `fastmcp generate-cli` turns any MCP server into a typed CLI; agent calls it via `shell` |
| `guardrails.py` | Hard-blocks catastrophic shell commands before approval even fires |
| `computer.py` | `ChromeDevToolsComputer` — full `AsyncComputer` impl over `chrome-devtools` CLI |

## Config Directories

```
~/.rival/
  sessions/     JSONL conversation history
  costs/        per-session token cost records
  jobs/         background job metadata (PID, status, output path)
  mcp/          user-scope MCP server CLIs + manifests
  skills/       user-scope skills
  agents/       user-scope sub-agent specs
  settings.json permissions (allow/deny tool lists)

{cwd}/.rival/   same structure, project-scoped (overrides user-scope on collision)
```

## Sub-agent Spec Format (`.rival/agents/<name>.md`)

```markdown
---
name: my-agent
description: what the orchestrator sees
model: grok-4-fast          # optional
tools: [read_file, shell]   # optional; default: read_file, search_in_files, get_file_tree, list_directory
skills: [skill-name]        # optional; injects skill catalog + activate_skill tool
mcp: [server-name]          # optional; injects MCP catalog, adds shell tool
max_turns: 20               # optional
persistent: false           # true = shared JSONLSession across invocations
---

System prompt for the sub-agent goes here.
```

## Skill Discovery Scan Order

Project overrides user; first-found wins within the same scope:
1. `{cwd}/.agents/skills/`
2. `{cwd}/.claude/skills/`
3. `{cwd}/.rival/skills/`
4. `~/.agents/skills/`
5. `~/.claude/skills/`
6. `~/.rival/skills/`

## Important Constraints

- The CLI command is `rival`, the package is `rival_code`, the config dir is `.rival/` — keep these distinct
- Never add "autonomous" to any description — it's a coding agent, not an autonomous agent
- `OpenAIResponsesModel` is required (not `OpenAIChatCompletionsModel`) — the Responses API is needed for `ComputerTool` and is supported by xAI and OpenAI
- Provider is configured via `RIVAL_API_KEY` / `RIVAL_BASE_URL` / `RIVAL_MODEL` env vars (or `--base-url` / `--model` CLI flags). Falls back to `OPENAI_API_KEY` and `XAI_API_KEY` for backwards compat
- `RIVAL.md` in the project root or `~/RIVAL.md` injects persistent instructions into the system prompt (same role as `CLAUDE.md` for Claude Code)
- Sessions are capped at 500 items via `SessionSettings(limit=500)` in `RunConfig`
- Web search results are cached 5 minutes at module level in `tools.py` — the cache persists across tool calls within a session
- Guardrails in `guardrails.py` hard-block catastrophic shell patterns even with `--no-approval`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/damionrashford)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/damionrashford)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## xrpl-mcp-server

> - Define durable repository instructions for coding agents working in this repo.

# AGENTS.md

## Purpose

- Define durable repository instructions for coding agents working in this repo.
- Keep long-lived project context here and in the repo-local skill at `.codex/skills/xrpl-mcp-server-maintainer/`.

## Repository Snapshot

- Project: `xrpl-mcp-server`
- Primary stack: `Python 3.10+`
- Runtime shape: single-package FastMCP server under `src/xrpl_mcp_server/`
- Main entrypoints: `python -m xrpl_mcp_server` and `xrpl-mcp-server`

## Commands

- Install dependencies: `python3 -m venv .venv && source .venv/bin/activate && python -m pip install -e ".[dev]"`
- Run the MCP server: `python -m xrpl_mcp_server`
- Run the MCP Inspector: `mcp dev src/xrpl_mcp_server/server.py:mcp --with-editable .`
- Build package artifacts: `python -m pip install build && python -m build`
- Run tests: `pytest`
- Run lint checks: `flake8 src tests && mypy src`
- Run formatters: `isort src tests && black src tests`
- Refresh the lockfile: `uv lock`

## Workflow Rules

- Read this file before making edits. For repo-specific architecture and backlog context, also use `.codex/skills/xrpl-mcp-server-maintainer/SKILL.md`.
- Keep the server `stdio`-first unless the user explicitly asks for another transport.
- Preserve MCP tool names and input arguments unless the user explicitly requests a breaking change.
- Prefer structured MCP outputs for successful tool calls and `ToolError` for failure paths.
- Treat `XRPL_NODE_URL` as mainnet by default. When touching `submit_transaction`, examples, or install docs, call out live-network implications.
- Use `SubmitOnly(tx_blob=...)` for signed blob submission with current `xrpl-py`.
- Keep tests alongside behavior changes. Code changes should normally include `pytest` coverage updates.

## Verification Expectations

- For Python code changes: run `pytest`.
- For packaging or dependency changes: also verify the documented startup paths still work.
- For skill edits: run `python -m pip install PyYAML && python /Users/lgcarrier/.codex/skills/.system/skill-creator/scripts/quick_validate.py .codex/skills/xrpl-mcp-server-maintainer`.
- If you cannot run a relevant command, say so clearly in the final handoff.

## Handoff Expectations

- Summarize behavior changes and impacted paths.
- Include the verification commands run and the outcomes.
- Call out network-risk changes, assumptions, and any follow-up work that remains.

---
> Source: [lgcarrier/xrpl-mcp-server](https://github.com/lgcarrier/xrpl-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->

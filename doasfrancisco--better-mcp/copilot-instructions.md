## better-mcp

> See [THOUGHTS.md](THOUGHTS.md) for lessons learned building MCP servers (FastMCP quirks, Gmail API patterns, tool design).

## Insights

See [THOUGHTS.md](THOUGHTS.md) for lessons learned building MCP servers (FastMCP quirks, Gmail API patterns, tool design).

## Hooks

See [HOOKS.md](HOOKS.md) for Claude Code hooks patterns (sound effects, skill hooks, Windows syntax).

## Old architecture

See [LEGACY.md](LEGACY.md) for the removed build pipeline (build-mcps, generate-mcp-config, config templates).

## Python MCPs

All Python MCP servers use **uv** for dependency management. Use `uv add <package>` to install dependencies (not pip). Each MCP has its own `pyproject.toml` and `.venv/`.

## WhatsApp MCP

Architecture pattern: **cache-first**. Always build logic that caches data locally before hitting the API. Every call goes through the WhatsApp Web bridge to the phone — excessive calls trigger rate limits or bans. Minimize API calls by caching contacts, chat lists, and other stable data to local JSON files.

## After finishing a task

Update [FUTURE.md](FUTURE.md) — remove completed items and add any new ideas or next steps that came up during the work.

## Commits

Use conventional commits. No co-authoring. ALWAYS stage with `git add .` — before committing, check `.gitignore` to verify no file that should not be added is missing from it.

Every message MUST answer both **what** changed and **why** it changed. A message that only describes the what is wrong.

- `feat:` for new functionality
- `fix:` for bug fixes
- `chore:` for everything else

Bad: `feat: add repo mapping` — missing the why.
Good: `feat: add repo mapping to centralize project navigation`

Bad: `fix: update query logic` — what query? why?
Good: `fix: deduplicate subsidiary query to avoid double-counting schedules`

Bad: `chore: update dependencies` — why now?
Good: `chore: bump django to 4.2.9 to patch CVE-2024-XXXXX`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/doasfrancisco)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/doasfrancisco)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

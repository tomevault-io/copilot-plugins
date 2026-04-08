## proforwarder-discord-bot

> **First action in every session:** Read `CLAUDE.md` in the project root. It is the single authority for all project rules — architecture, config patterns, database schema, key patterns, and gotchas. Everything below is Codex-specific supplemental information.

# AI Agent Instructions

## Start Here

**First action in every session:** Read `CLAUDE.md` in the project root. It is the single authority for all project rules — architecture, config patterns, database schema, key patterns, and gotchas. Everything below is Codex-specific supplemental information.

- **Claude Code**: `CLAUDE.md` is loaded automatically via project instructions.
- **Codex / other agents**: You **MUST** read `CLAUDE.md` with your file reading tool before taking any action.

**Fallback only** (if `CLAUDE.md` is missing or unavailable):
- Architecture & patterns: `README.md`
- Tool selection: `AI_TOOLS.md`
- Design docs: `Documentations/` (historical reference)

## Codex MCP Tool Namespace

All MCP tools are accessed through **MCP Router**. In Codex, the server name is normalized to `mcp_router`, so tools appear as `mcp__mcp_router__*`. Full tool reference with all parameters: `AI_TOOLS.md`.

**Quick decision order:**

| Task | Use MCP | Instead of |
|---|---|---|
| Codebase navigation | `roam_*` tools | Manual grep/glob/file reading |
| Large output commands | `ctx_execute` / `ctx_batch_execute` | Bash (which floods context) |
| Symbol search | `roam_search_symbol` / `roam_batch_search` | grep for function names |
| Impact analysis | `roam_impact` / `roam_preflight` | Reading files to guess dependencies |
| Library docs | `resolve-library-id` + `get-library-docs` | Web search for Node.js/Discord.js docs |

**Roam quick decision:**

| Situation | MCP Tool |
|---|---|
| First time in this repo | `roam_understand` then `roam_explore` |
| Need to modify a symbol | `roam_preflight` or `roam_prepare_change` |
| Debugging a failure | `roam_diagnose` or `roam_diagnose_issue` |
| Need files to read | `roam_context` (files + line ranges) |
| Find a symbol | `roam_search_symbol` or `roam_batch_search` (up to 10) |
| What breaks if I change X? | `roam_impact` |
| Pre-PR check | `roam_pr_risk` or `roam_review_change` |
| After making changes | `roam_diff` (blast radius of uncommitted changes) |

**Context Mode:** Use `ctx_execute` / `ctx_batch_execute` instead of Bash for commands with large output (>20 lines).

**GitHub:** Use `gh` CLI (installed globally), NOT MCP.

## Project-Specific Notes

- **No test suite** — verify changes via web admin UI or bot behavior. No `npm test` command exists.
- **No linter** — no ESLint, Prettier, or similar configured.
- **Single entry point** — `index.js` boots everything (Discord client, database, web server, reader bot).
- **All web API routes are in one file** — `web/server.js` (3,600+ lines). Use line ranges.
- **Frontend is vanilla JS** — no React, no build tools. Each tab is a separate `.js` file in `web/public/`.
- **Config is a JS module** — `config/config.js` reads `.env` and exports a plain object. Not JSON.
- **Runtime JSON files** — `data/forwardConfigs.json`, `data/autoPublish.json`, `data/cachedInvites.json` are written by the bot at runtime. Do NOT edit while running.

## Troubleshooting

- **mcp-router not loading** → Check `~/.codex/config.toml` has the `mcp-router` entry.
- **Docker MCP tools not responding** → Docker Desktop may need to be running.
- **Roam not found** → `pip install roam-code`
- **Roam index stale** → `roam index` (incremental) or `roam index --force` after major refactors.
- **Roam times out** → Retry with a narrower call (`roam_context`, `roam_search_symbol`); if still failing, use `ctx_batch_execute` and `rg`.
- **context-mode issues** → Run `ctx_doctor` to diagnose.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fundryi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fundryi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

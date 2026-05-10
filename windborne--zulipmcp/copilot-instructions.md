## zulipmcp

> **Public open-source repo.** All code, commits, and PRs are visible to the world.

# zulipmcp

**Public open-source repo.** All code, commits, and PRs are visible to the world.

## Rules

- Nothing WindBorne-specific. No internal URLs, emails, Zulip instances, or infra references.
- No internal context in PRs, commits, or comments.
- Elegant, general code. Type hints everywhere. Minimal dependencies.

## Architecture

- **`core.py`** — Zulip API wrappers. No MCP dependency. Returns Python objects.
- **`mcp.py`** — MCP tool layer + `SessionState`. Thin wrappers over `core.py`. Hooks system lives here.

Separation is load-bearing: `core.py` must stay MCP-agnostic so it works as a standalone library import.

## Gotchas

- **`listen()` is async, everything else is sync.** It runs `get_events()` in a thread executor to interleave MCP keepalive pings. Don't make other tools async — FastMCP handles sync tools fine.
- **Event queue narrow does NOT filter reactions.** `is_dismiss_reaction()` receives reactions from all streams — it fetches the reacted-on message to verify stream/topic. Don't remove that check.
- **`_session` is module-level singleton.** One session per process. Don't add multi-session — MCP is one-client-per-server.
- **`reply()` checks for missed messages before sending.** The `last_seen_message_id` bookkeeping is subtle — trace it carefully before changing.
- **Two dismiss-check paths exist intentionally.** `listen()` catches via events, `reply()` via REST poll. Both needed — user might react during tool execution (not listening).
- **Private stream security is asymmetric on purpose.** Unset `BOT_ALLOWED_PRIVATE_STREAMS` = no access (default-deny). Unset `BOT_ALLOWED_WRITE_STREAMS` = all writes allowed (backwards-compat). Don't "fix" the asymmetry.
- **`configure()` must be called before `run_server()`.** `run_server()` may auto-init a session that reads hook state.

---
> Source: [windborne/zulipmcp](https://github.com/windborne/zulipmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

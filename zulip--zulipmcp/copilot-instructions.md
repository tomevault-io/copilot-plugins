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
- **`agent_backends.py`** — Claude/Codex/OpenCode command builders for the listener. No Zulip API dependency.
- **`hermes_plugin/zulip/`** — Native Hermes gateway transport. It does not use MCP session state.

Separation is load-bearing: `core.py` must stay MCP-agnostic so it works as a standalone library import.

## Gotchas

- **`listen()` is async, everything else is sync.** It runs `get_events()` in a thread executor to interleave MCP keepalive pings. Don't make other tools async — FastMCP handles sync tools fine.
- **Event queue narrow does NOT filter reactions.** `is_dismiss_reaction()` receives reactions from all streams — it fetches the reacted-on message to verify stream/topic. Don't remove that check.
- **`_session` is module-level singleton.** One session per process. Don't add multi-session — MCP is one-client-per-server.
- **`reply()` checks for missed messages before sending.** The `last_seen_message_id` bookkeeping is subtle — trace it carefully before changing.
- **Send tools enforce `MAX_MESSAGE_LENGTH`.** Zulip silently truncates over-limit messages and still reports success — the guard returns an error to the caller instead. It measures the *normalized* form (post `normalize_zulip_markdown()`), since normalization can grow content. Override with `ZULIP_MAX_MESSAGE_LENGTH` for non-default realm caps.
- **Two dismiss-check paths exist intentionally.** `listen()` catches via events, `reply()` via REST poll. Both needed — user might react during tool execution (not listening).
- **Interrupt-file reads must go through `_consume_interrupt_file()`.** It claims via atomic rename before reading, so a writer replacing the file mid-consume can't have its content deleted unread — and it decodes with `errors="replace"` so a bad-bytes file can't wedge every future poll. Don't inline a read+delete elsewhere.
- **Private stream security is asymmetric on purpose.** Unset `BOT_ALLOWED_PRIVATE_STREAMS` = no access (default-deny). Unset `BOT_ALLOWED_WRITE_STREAMS` = all writes allowed (backwards-compat). Don't "fix" the asymmetry.
- **`configure()` must be called before `run_server()`.** `run_server()` may auto-init a session that reads hook state.
- **Codex MCP config is not `.mcp.json` native.** The listener translates `.mcp.json` into Codex `-c mcp_servers...` overrides. Keep secrets in env/header fields; env refs in command/args/cwd/url must fail closed to avoid argv leaks.
- **OpenCode MCP config is not `.mcp.json` native.** The listener translates `.mcp.json` into inline JSON via `OPENCODE_CONFIG_CONTENT`. Header values are embedded directly (no env-var indirection like Codex). Env refs in command/args/url are rejected.
- **`core.py` send paths (`send_message`/`send_direct_message`/`edit_message`) rewrite markdown via `normalize_zulip_markdown()`.** Blank lines are injected before tables and bold/link combos Zulip breaks on are rewritten. Fenced/indented code is exempt from both fixes; inline code spans are exempt only for the bold/link rewrite. The Hermes gateway adapter sends via the raw client and is NOT normalized. `ZULIPMCP_MARKDOWN_AUTOFIX=0` disables all of it.
- **`remove_reaction()` needs explicit `reaction_type` for custom emoji.** Zulip's DELETE reactions endpoint defaults `reaction_type` to `unicode_emoji` — custom emoji removals fail silently without it. The listen indicator caches `_listen_reaction_type` alongside `_listen_emoji`; always pass both when removing.

---
> Source: [zulip/zulipmcp](https://github.com/zulip/zulipmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

## phoenix

> An MCP server + Claude Code hooks system for persistent AI agent memory.

# Phoenix Memory — project rules

## What this project is
An MCP server + Claude Code hooks system for persistent AI agent memory.
Zero token overhead: hooks log turns automatically, the model only calls tools for intentional reads/writes.

## Architecture
- `server.py`      — 3 MCP tools: initialize_session, create_manual_checkpoint, get_resume_packet
- `hook_logger.py` — phoenix-log CLI wired to PostToolUse and Stop hooks
- `database.py`    — SQLite backend, thread-safe (WAL mode)

## Rules
- Never add commit_log back as an MCP tool — logging is the hook's job, not the model's.
- Never import anthropic or tiktoken — this package has no AI dependency.
- Hooks must never crash Claude Code: always catch exceptions and exit 0.
- Keep hook_logger.py read-only toward the DB (write_log_entry only, never delete).
- Database writes are always atomic (committed immediately after each INSERT).
- The hook_logger reads transcript_path from the Stop event — never assume its format, handle both JSONL and JSON gracefully.

---
> Source: [khangbdd/phoenix](https://github.com/khangbdd/phoenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->

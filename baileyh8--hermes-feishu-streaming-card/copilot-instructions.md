## hermes-feishu-streaming-card

> A **sidecar-only** plugin that adds Feishu streaming card messages to

# Agents Guide — hermes-feishu-streaming-card

## What this is

A **sidecar-only** plugin that adds Feishu streaming card messages to
[Hermes Agent](https://github.com/NousResearch/hermes-agent).  Patches
Hermes's `gateway/run.py` at install time and runs a separate HTTP sidecar.

Active source: `hermes_feishu_card/`.  **`legacy/` is V2 archive — never edit.**

## Development commands

```bash
pip install -e ".[test]"               # from repo root
python -m pytest -q                     # full suite (CI runs exactly this)
python -m pytest tests/unit/test_config.py::test_load_defaults -q  # one test
```
No linter, typechecker, or formatter configured.  CI workflow = `pytest -q`.

## Never edit Hermes directly

The **patcher** (`install/patcher.py`) is the only code that touches
`gateway/run.py`.  It uses AST parsing to find `_handle_message_with_agent`,
inserts 5 marker-wrapped hook blocks, creates SHA256 manifests and backups.
Corrupt markers → `ValueError("corrupt patch markers")`.  Changed files →
refuses restore.

## Hook runtime uses `locals()` — fragile

`hook_runtime.py` functions extract data from the caller's `locals()` dict.
Field names in `build_event()` / `_event_data()` **must match Hermes variable
names**: `source`, `event`, `response`, `agent_result`, `_response_time`,
`event_message_id`, `_loop_for_step`, `_run_still_current`.  If Hermes
renames these, the hook breaks silently.

## Sidecar is ephemeral

`CardSession` objects live in `request.app[SESSIONS_KEY]` (plain dict keyed
by `_session_key(event)` in multi-profile mode, `message_id` otherwise).  No
persistence.  Each session has its own `asyncio.Lock`.  Terminal events
retry 3× with exponential backoff (1s, 2s, 4s).  Non-terminal update failures
are silently ignored (next event retries).

## Message ID fallback — don't touch

Hermes doesn't always provide `message_id`.  The system in
`hook_runtime.py` (`_fallback_message_id`, `_ACTIVE_FALLBACK_MESSAGE_IDS`,
`created_at_lifecycle_token`) handles dedup across parallel sessions.  Don't
simplify it unless you fully understand the lifecycle race conditions.

## Key constraints

- Hermes ≥ `v2026.4.23` (checked by `detect_hermes()`)
- `gateway/run.py` must not be a symlink
- `UPDATE_MIN_INTERVAL_SECONDS = 0.5` in `server.py`
- Tenant token cached for `expire - 60` seconds
- `hermes_feishu_card` must be importable inside Hermes's Python environment

## 语言约定

思考输出用中文。字段名、变量名、函数名、专用名词、工具名保持英文。

## Release checklist

1. Bump version in `pyproject.toml` and `hermes_feishu_card/__init__.py`
2. Update `CHANGELOG.md`, `README.md`, `README.en.md`, `config.yaml.example`, `TODO.md`
3. `python -m pytest -q` → must be all green
4. `git tag -a vX.Y.Z` AND `gh release create vX.Y.Z` — **both required**, not just tag
5. GitHub Release notes from CHANGELOG with `## VX.Y.Z — YYYY-MM-DD`

## Doc test warning

`tests/unit/test_docs.py` uses **exact string matching** (including `。`).  If
you change `TODO.md`, verify every string the tests assert on still exists.
Otherwise 5 doc tests fail and you'll waste time hunting Chinese punctuation.

---
> Source: [baileyh8/hermes-feishu-streaming-card](https://github.com/baileyh8/hermes-feishu-streaming-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

## claude-swiftbar

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A macOS SwiftBar / xbar menu bar plugin (`plugins/claude-usage.5m.py`) that displays Claude Code session usage, rate-limit utilisation, live context-window fill, and a compact/clear recommendation. It reads credentials from the macOS Keychain and calls `api.anthropic.com/api/oauth/usage`. Zero third-party dependencies — stdlib only, enforced by CI.

## Commands

```bash
# Install dev tools and activate the pre-push hook (one-time per clone)
pip install pytest ruff mypy bandit
git config core.hooksPath .githooks

# Run all tests
python -m pytest tests/ -v

# Run a single test class or function
python -m pytest tests/test_plugin.py::TestSafeMenuText -v
python -m pytest tests/test_plugin.py::TestSafeMenuText::test_pipe_replaced -v

# Lint and format check
ruff check plugins/ tests/
ruff format plugins/ tests/      # auto-fix
ruff format --check plugins/ tests/  # check only (CI mode)

# Type check (strict)
mypy plugins/claude-usage.5m.py --ignore-missing-imports --strict

# Security scan
bandit -r plugins/ -ll -ii
```

## Architecture

**Single-file plugin:** All logic lives in `plugins/claude-usage.5m.py`. The filename prefix `claude-usage.5m` is the SwiftBar polling interval (5 minutes). Renaming it changes the interval.

**Data flow:**

1. `load_token()` — reads the OAuth token from macOS Keychain via `/usr/bin/security find-generic-password`. Never touches a file, never logs the token.
2. `fetch_usage(token)` — calls `USAGE_API_URL` with the token. On HTTP 429, stores a `backoff_until` epoch in the cache and skips future calls until that time passes.
3. `parse_latest_session()` — finds the most-recently-modified `~/.claude/projects/**/*.jsonl` file and sums token counts from `assistant` message entries. Bounded by `MAX_JSONL_BYTES` (50 MB) and `MAX_JSONL_LINES` (50 000).
4. `recommend(s_pct, ctx_pct, had_compaction)` — computes a status emoji + advice string from session utilisation and context fill percentage.
5. `_print_all()` / `_print_body()` — formats xbar/SwiftBar menu output (pipe-separated key=value attributes on each line).
6. Cache at `~/.cache/claude-swiftbar/last_good.json` — stores only usage percentages and reset timestamps (never tokens). Written atomically with `os.replace` and `0600` permissions. Symlinks are refused.

**Testing approach:** `tests/test_plugin.py` imports the plugin by `exec()`-ing the source into a module to avoid executing `main()`. All tests exercise pure functions; no network or Keychain calls. CI runs on Python 3.11, 3.12, and 3.13 on macOS.

**CI checks** (`.github/workflows/ci.yml`): ruff lint, ruff format, mypy strict, bandit, hardcoded-secret grep, third-party import AST check, install.sh absolute-path check, shellcheck. All checks are mirrored in `.githooks/pre-push`, which runs automatically before every `git push` once `git config core.hooksPath .githooks` is set.

## Key constraints to preserve

- **stdlib only** — no `pip install` at runtime. The CI AST check enforces the allowed import list.
- **No file writes except the cache** — the plugin must never write to, rename, or delete JSONL session files.
- **All untrusted text must pass through `safe_menu_text()`** before being printed to menu output, to prevent xbar pipe-injection, control characters, and excessively long strings.
- **`SECURITY_BIN = "/usr/bin/security"`** — always use this constant (absolute path, `shell=False`) for Keychain access.
- **Line length:** 100 characters (`pyproject.toml`).
- **Runtime target:** Python 3.9.6+ (ruff `target-version = "py39"` enforces this). mypy uses `python_version = "3.10"` because mypy dropped 3.9 target support — this does not change the runtime requirement.

---
> Source: [jdetras/claude-swiftbar](https://github.com/jdetras/claude-swiftbar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->

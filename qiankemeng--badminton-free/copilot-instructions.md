## badminton-free

> This repository is a flat Python CLI automation project. `badminton.py` is the interactive entry point. `booking_core.py` contains the shared booking and query requests, and `config_manager.py` reads and writes the local user config at `~/.badminton_config.json`. Other root-level scripts handle focused tasks: `auto_grab.py` for scheduled booking, `capture_proxy.py` for `mitmproxy` capture, and `update_token.py`, `my_bookings.py`, `invite_partner.py`, `complete_booking.py`, and `correct_booking.py` for one-off utilities. Runtime files such as `headers.json` and `token.txt` should be treated as local state.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a flat Python CLI automation project. `badminton.py` is the interactive entry point. `booking_core.py` contains the shared booking and query requests, and `config_manager.py` reads and writes the local user config at `~/.badminton_config.json`. Other root-level scripts handle focused tasks: `auto_grab.py` for scheduled booking, `capture_proxy.py` for `mitmproxy` capture, and `update_token.py`, `my_bookings.py`, `invite_partner.py`, `complete_booking.py`, and `correct_booking.py` for one-off utilities. Runtime files such as `headers.json` and `token.txt` should be treated as local state.

## Build, Test, and Development Commands
- `python3 -m venv .venv && source .venv/bin/activate` creates a local virtual environment.
- `pip install -r requirements.txt` installs the current dependencies: `requests`, `mitmproxy`, and `schedule`.
- `python3 badminton.py` launches the main terminal menu for booking, settings, and token refresh.
- `python3 auto_grab.py` starts the scheduled booking loop and performs one immediate test attempt.
- `mitmdump -s capture_proxy.py -p 8080` captures DingTalk requests so headers and tokens can be refreshed.
- `python3 update_token.py` extracts the newest bearer token from the capture log.

## Coding Style & Naming Conventions
Use Python 3 with 4-space indentation. Follow existing naming patterns: `snake_case` for functions and variables, `UPPER_SNAKE_CASE` for module constants, and short module docstrings where helpful. Keep scripts task-focused, and move shared API or config logic into `booking_core.py` or `config_manager.py` instead of duplicating request code. Preserve the existing Chinese CLI copy unless you are intentionally revising the full interaction flow.

## Testing Guidelines
There is no automated test suite yet. Validate changes with manual smoke tests before opening a PR:
- `python3 badminton.py` for menu flow and config persistence.
- `python3 my_bookings.py` to confirm token and header handling still work.
- `mitmdump -s capture_proxy.py -p 8080` when changing auth capture behavior.
Record the scenario tested and any notable API responses in the PR description.

## Commit & Pull Request Guidelines
Recent commits follow short Conventional Commit prefixes such as `feat:` and `fix:`, usually with concise Chinese summaries, for example `feat: 添加候补预约功能（多场地轮换）`. Keep commits scoped to one behavior change. PRs should explain the booking scenario affected, list manual verification steps, note any config or secret-handling changes, and include terminal output or screenshots if the CLI flow changed.

## Security & Configuration Tips
Do not commit live tokens, capture logs, phone numbers, or personal `openid` values. Scrub `headers.json` before sharing, and prefer storing user-specific values in `~/.badminton_config.json` rather than hardcoding them in scripts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qiankemeng)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qiankemeng)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

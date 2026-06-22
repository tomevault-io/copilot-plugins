## codex-mate

> This repository contains Codex Mate, a local companion launcher for the Codex App. It starts Codex with a debug port, runs a helper server, and injects UI enhancements without modifying the installed Codex App bundle.

# Codex Mate Contributor Notes

This repository contains Codex Mate, a local companion launcher for the Codex App. It starts Codex with a debug port, runs a helper server, and injects UI enhancements without modifying the installed Codex App bundle.

## Project Layout

- `codex_mate/cli.py`: command-line entry points for launch, setup, update, diagnostics, watcher, and history sync.
- `codex_mate/launcher.py`: starts Codex and injects the renderer script.
- `codex_mate/cdp.py`: Chrome DevTools Protocol helpers and script injection.
- `codex_mate/helper_server.py`: local HTTP helper used by injected UI actions.
- `codex_mate/inject/renderer-inject.js`: browser-side UI injection.
- `codex_mate/history_sync.py`: local Codex history provider/model alignment and sidebar index repair.
- `codex_mate/windows_installer.py` and `codex_mate/macos_installer.py`: platform launcher/install helpers.
- `codex_mate/watcher.py`: optional transparent takeover of normally launched Codex.
- `tests/`: pytest coverage for CLI, installers, updater, injection, history sync, and release workflow.

## Development Commands

- Run the full test suite: `pytest -q`
- Run focused tests: `pytest -q tests/test_history_sync.py tests/test_launcher_cli.py`
- Check package metadata: `python -m codex_mate doctor --json`
- Check local history state: `python -m codex_mate history-status --json`
- Force history alignment: `python -m codex_mate history-sync --json`
- Launch from source: `python -m codex_mate launch`

## Implementation Guidelines

- Keep edits scoped. This project touches local Codex state, shortcuts, and startup flows, so small changes with focused tests are preferred.
- Do not modify installed Codex App files. Codex Mate should continue to work through external launching, CDP injection, and helper APIs.
- Preserve explicit user escape hatches. For example, `launch --no-history-sync` should remain available even when default launch behavior changes.
- Prefer condition checks before state writes. Startup should avoid unnecessary database rewrites, session rewrites, and backup creation when history is already aligned.
- Keep Windows and macOS behavior in sync where practical, but respect each platform's launcher and permission model.
- When changing launch, watcher, installer, updater, or history behavior, add or update tests in the matching `tests/test_*.py` file.

## History Sync Notes

History sync exists to repair local sidebar visibility after switching Codex account, provider, or model. The normal launch path should first perform a lightweight status check and only run full sync when existing local threads mismatch the current `config.toml` provider/model.

Important expectations:

- `history-status` is read-only.
- `history-sync` is the manual force-sync command.
- Startup sync should skip when provider/model already match.
- Full sync creates backups under `~/.codex/codex_mate_history_backups`.
- Full sync may update `state_5.sqlite`, `sessions/**/rollout-*.jsonl`, `session_index.jsonl`, and `.codex-global-state.json`.

## Release Process

- Bump both `codex_mate/__init__.py` and `pyproject.toml`.
- Run `pytest -q` before publishing.
- Commit the version bump and code changes together when they are part of the same release.
- Push `main`, then create a GitHub release tag such as `v1.1.18`.
- The `.github/workflows/release-assets.yml` workflow builds and uploads:
  - `CodexMate.zip`
  - `CodexMate-windows.zip`
  - `CodexMate-macos.zip`

## Git Hygiene

- Check `git status --short` before editing and before finishing.
- Never revert unrelated user changes.
- Avoid destructive commands such as `git reset --hard` or `git checkout --` unless the user explicitly asks.
- If release assets or workflow runs are involved, confirm the final GitHub Actions status before reporting completion.

---
> Source: [serein431/Codex-Mate](https://github.com/serein431/Codex-Mate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->

## ghostty-workspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A macOS utility that launches and manages a multi-tab Ghostty terminal workspace from a YAML config file. It generates AppleScript that drives Ghostty's scripting API and System Events to create tabs, set titles, configure splits, and run startup commands.

## Commands

```bash
# Run tests (requires PyYAML)
python3 tests.py

# Dry run (validate config, print planned actions)
python3 ghostty-workspace.py --dry-run

# Launch workspace
python3 ghostty-workspace.py -c ghostty-workspace.yaml

# Debug: print generated AppleScript
python3 ghostty-workspace.py --print-script
```

## Architecture

**Single-script design** — `ghostty-workspace.py` is the entire application.

Flow: YAML config → `parse_tabs()` → `build_payload()` → `to_applescript()` injects a native AppleScript record literal into `APPLE_SCRIPT` template → `run_osascript()` pipes it to `osascript`.

Key design decisions:
- Payload is injected as a native AppleScript record literal (via `to_applescript()`), not JSON — AppleScript has no JSON parser
- Working directory uses Ghostty's `initial working directory` surface config, NOT `cd` in `initial input` — this is a tested invariant (see `TestAppleScriptInjection`)
- Tab titles are set via Ghostty's `prompt_tab_title` action + System Events UI automation (requires Accessibility permission)
- Tab ordering uses Ghostty's `move_tab:-1` action to slide tabs into position
- `die()` raises `SystemExit` for all validation errors — tests check for this

## Config Format (ghostty-workspace.yaml)

Top-level keys: `window` (settings like `shell`, `tab_position`, `reuse_existing_tabs`, `always_new`) and `tabs` (list). Each tab requires `key` (unique). Optional: `title` (unique, triggers Accessibility-based rename), `working_dir`, `command`, `shell`, `split` (with `direction`, `ratio`, `second_pane_command`), `focus`, `reuse_if_exists`, `enabled`.

## Dependencies

- Python 3.9+, PyYAML
- macOS with Ghostty (AppleScript support) and Accessibility enabled for System Events

---
> Source: [manonstreet/ghostty-workspace](https://github.com/manonstreet/ghostty-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

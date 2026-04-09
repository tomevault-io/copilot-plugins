## claude-dolphin-konsole-actions

> KDE Dolphin service menu actions that launch Claude Code in various window layouts. Target environment is Ubuntu 25.10 with KDE Plasma 6.4.5 on Wayland.

# Claude Dolphin & Konsole Actions

## Project Overview

KDE Dolphin service menu actions that launch Claude Code in various window layouts. Target environment is Ubuntu 25.10 with KDE Plasma 6.4.5 on Wayland.

## Key Paths

- **Service menus install location:** `~/.local/share/kio/servicemenus/`
- **Actions source:** `actions/<action-id>/` — each contains a `.desktop` file, `install.sh`, and `uninstall.sh`
- **Spec:** `planning/SPEC.md`
- **Private notes (not for publishing):** `private`

## Conventions

- Each action gets its own subdirectory under `actions/`
- `.desktop` files follow the KDE 6 service menu format (`Type=Service`, `MimeType=inode/directory;`)
- Install scripts copy the `.desktop` file to the service menus directory; uninstall scripts remove it
- Multi-window layouts need a wrapper script (Wayland constraint) — stored alongside the `.desktop` file

## Current Status

- **Action 1 (Open In Claude):** Complete and installed on this machine
- **Actions 2-5:** Not yet built — see `planning/SPEC.md`

## Notes

- This is a work in progress
- The `private` file contains raw planning notes and should not be included in documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielrosehill)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielrosehill)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

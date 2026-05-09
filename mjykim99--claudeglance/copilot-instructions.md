## claudeglance

> This repository contains `ClaudeGlance`, a macOS status HUD for Claude Code sessions. Agents working here should optimize for safe, targeted changes and preserve the existing runtime path from Claude hooks to the HUD.

# AGENTS Guide

## Purpose

This repository contains `ClaudeGlance`, a macOS status HUD for Claude Code sessions. Agents working here should optimize for safe, targeted changes and preserve the existing runtime path from Claude hooks to the HUD.

## Repository Map

- `ClaudeGlance/ClaudeGlanceApp.swift`: app entry, menu bar app lifecycle, settings window, hook installation and diagnostics
- `ClaudeGlance/Services/IPCServer.swift`: Unix socket and HTTP listener, health checks, reconnect, fallback ports
- `ClaudeGlance/Services/SessionManager.swift`: session state machine, stats, cleanup, event filtering
- `ClaudeGlance/Models/SessionState.swift`: session models and Claude hook payload decoding
- `ClaudeGlance/Views/`: HUD and session presentation
- `Scripts/`: repo-level utility scripts
- `ClaudeGlance/Scripts/`: scripts bundled into the app

## Required Context Before Editing

Read the relevant files before changing behavior. In particular:

- For hook installation or diagnostics: read `ClaudeGlance/ClaudeGlanceApp.swift`
- For session behavior or status changes: read `ClaudeGlance/Services/SessionManager.swift`
- For transport or port changes: read `ClaudeGlance/Services/IPCServer.swift`
- For hook payload schema changes: read `ClaudeGlance/Models/SessionState.swift` and the reporter shell script

## Editing Expectations

- Keep changes minimal and local.
- Preserve existing behavior unless the task asks for a behavioral change.
- Do not rewrite the app architecture without strong justification.
- Do not remove compatibility paths without confirming they are truly unused.
- Update documentation if a user-facing workflow changes.

## High-Risk Areas

- Hook auto-install and settings repair in `ClaudeGlanceApp.swift`
- Session state transitions and timeout logic in `SessionManager`
- Socket path `/tmp/claude-glance.sock`
- HTTP port selection `19847...19857`
- Shadowed project settings detection for local `.claude/settings.json`
- Script duplication between `Scripts/` and `ClaudeGlance/Scripts/`

## Script Policy

If a task changes any of these files:

- `claude-glance-reporter.sh`
- `install.sh`
- `uninstall.sh`
- `build-dmg.sh`

check whether the corresponding file in the other scripts directory also needs the same change. The app installs the bundled script from `ClaudeGlance/Scripts/`, so mismatches are easy to introduce.

## Validation

Use the lightest verification that gives confidence:

```bash
xcodebuild -scheme ClaudeGlance -configuration Release
```

For script changes, also inspect shell syntax and the call sites that reference the script.

## Communication

When reporting work:

- mention the user-visible outcome first
- mention any unverified areas explicitly
- call out script-sync or hook-config risk when relevant

---
> Source: [MJYKIM99/ClaudeGlance](https://github.com/MJYKIM99/ClaudeGlance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

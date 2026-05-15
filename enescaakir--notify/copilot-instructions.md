## notify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

Claude Code plugin for Windows — fires toast notifications + audio on hook events. No build step. No tests. Edit files directly.

## Architecture

| File | Role |
|------|------|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, author) |
| `hooks/hooks.json` | Hook event bindings |
| `hooks/notify.ps1` | Generic notifier — accepts `-Title`, `-Message`, `-Sound` params; used by `Stop` |
| `hooks/notify-permission.ps1` | Permission notifier — writes marker file, starts 3s delayed notification |
| `hooks/notify-post.ps1` | Post-tool cleanup — removes marker file to cancel pending notification |
| `assets/claude-logo.jpg` | App logo shown in toasts |

## Hook Event Logic

**`Stop`** → `notify.ps1` with fixed title/message/sound. Simple.

**`PreToolUse`** → `notify-permission.ps1`:
1. Reads stdin JSON → extracts `tool_name`
2. Writes marker file `%TEMP%\claude-perm-<tool>.pending`
3. Starts background PS: waits 3s, checks marker still exists, fires toast if so

**`PostToolUse`** → `notify-post.ps1`:
1. Reads `tool_name` from stdin
2. Removes marker file → cancels pending notification

Auto-approved tools: `PostToolUse` removes marker before 3s → no toast. Tools needing permission: user sees prompt while 3s expires → toast fires.

## Dependencies

- **BurntToast** PowerShell module: `Install-Module BurntToast -Scope CurrentUser`
- Windows only (WPF `PresentationCore` + Windows toast APIs)
- Built-in Windows sounds at `C:\Windows\Media\` must exist

## Extending

Add new event to `hooks/hooks.json` under the hook name (e.g. `UserPromptSubmit`). Call `notify.ps1` with custom params, or write a new script for custom logic. `${CLAUDE_PLUGIN_ROOT}` resolves to plugin root at runtime.

Active hooks: `PreToolUse`, `PostToolUse`, `Stop`. Do not duplicate these.

---
> Source: [enescaakir/notify](https://github.com/enescaakir/notify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->

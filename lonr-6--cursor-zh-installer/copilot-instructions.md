## cursor-zh-installer

> This repository is an **agent-friendly Windows Cursor Chinese enhancement toolkit**.

# Agent Install Guide

This repository is an **agent-friendly Windows Cursor Chinese enhancement toolkit**.

It is **not**:

- a Cursor Marketplace plugin
- a standalone skill
- a bundled Cursor binary

## Supported environment

- OS: Windows
- Shell: PowerShell
- Runtime: Node.js in `PATH`
- Target: Installed Cursor desktop app

## Required install flow

When an agent is asked to install this toolkit, follow this exact order:

1. Confirm Windows and PowerShell are available.
2. Confirm `node` is available in `PATH`.
3. Detect the Cursor install directory.
   - First honor a user-provided install path.
   - Then check common candidates such as `%LOCALAPPDATA%\\Programs\\Cursor`.
4. Run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install.ps1
```

5. Run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\doctor.ps1
```

6. Report:
   - detected Cursor path
   - Cursor version
   - language pack compatibility
   - whether shortcut creation succeeded
   - how to start Cursor with the Chinese wrapper

## Required uninstall flow

Use:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\uninstall.ps1
```

The uninstall flow must not delete user profile data, history, or workspace state.

## Safety boundaries

- Do not distribute or commit `cursor/`, `state/`, backups, logs, screenshots, or user data.
- Do not upload modified Cursor binaries.
- Do not claim official affiliation with Cursor.
- Do not rewrite user rules, skill IDs, model IDs, or other behavior-critical identifiers.

---
> Source: [lonr-6/cursor-zh-installer](https://github.com/lonr-6/cursor-zh-installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

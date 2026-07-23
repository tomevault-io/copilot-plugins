## runcat365

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## You Must

- Respond in the same language the user is writing in.
- Commit to Git only when instructed.
- If multiple requirements are given in a single instruction, divide the commits into appropriate sizes/granularities.
- When creating a pull request, use the `.claude/skills/create-pr` skill so the project's `.github/pull_request_template.md` is followed.

## Project Overview

Windows Forms application (.NET 9.0 / C#) distributed via the Microsoft Store. Runs as a system tray icon whose animation reflects system load.

**Top-level layout:**

- `RunCat365.sln` — Main solution
- `RunCat365/` — Main application project
- `WapForStore/` — Windows Application Packaging project for the Microsoft Store

**Build:** Open `RunCat365.sln` in Visual Studio. Supported platforms: x64, x86, ARM64. Target framework: .NET 9.0 (Windows 10.0.26100.0).

**Releasing — version numbers must be updated in two places:**

1. `RunCat365/RunCat365.csproj` — `<Version>X.Y.Z</Version>` (3-digit)
2. `WapForStore/Package.appxmanifest` — `Version="X.Y.Z.0"` in `<Identity>` (4-digit)

## Architecture (patterns, not file lists)

- **Entry point:** `Program.cs` hosts the tray application context that owns the timers, the tray icon, and the context menu.
- **Repository pattern for system metrics:** Each metric has a `*Repository` class that produces a matching `*Info` struct (CPU / GPU / memory / storage / network / temperature). Add a new metric by adding a paired repository and info struct.
- **Animation loop:** A 1-second fetch timer refreshes the `*Info` structs; a separate animate timer advances frames at a rate derived from the `SpeedSource` setting. Theme-aware icon recoloring lives in `BitmapExtension`.
- **Custom runners:** User-defined animations are loaded from `profiles.json` and managed by the `CustomRunner*` classes; the active profile is stored in the `CustomRunnerName` setting.
- **EndlessGame:** Mini-game living in `EndlessGameForm` and its collaborator classes.
- **Settings & resources:** Persisted user preferences in `Properties/UserSettings.settings`; embedded images/icons in `Properties/Resources.resx`.

## Localization

- Localized strings live in `Properties/Strings.resx` (English default) and sibling `Properties/Strings.{locale}.resx` files. Any new string must be added to every variant.
- Per-locale font and culture settings live in `SupportedLanguage.cs`.
- When adding a brand-new locale (not just new strings), follow the `.claude/skills/add-locale` skill.

## Coding Rules

General contribution rules — indentation style, `var` usage, single-change PRs, English-only code, license, and the rest — are documented in [`CONTRIBUTING.md`](CONTRIBUTING.md). Follow those first.

Project-specific additions:

- Do not write comments within the source code. Use names that make intent obvious.
- Write abbreviations such as URL or ID in either all lowercase or all uppercase (do not use Upper Camel Case for them).
- Do not use abbreviations like `img` for `image` or `cnt` for `count`.

---
> Source: [runcat-dev/RunCat365](https://github.com/runcat-dev/RunCat365) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->

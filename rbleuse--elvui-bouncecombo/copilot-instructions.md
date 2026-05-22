## elvui-bouncecombo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ElvUI_BounceCombo** is a World of Warcraft addon (WoW Anniversary/TBC client, Interface 20505) that adds a bounce animation to ElvUI's combo point display when combo points are gained.

- **Author:** Spanja
- **Version:** 1.1
- **Dependency:** ElvUI (required)

## Architecture

The addon is a single-file ElvUI plugin ([ElvUI_BounceCombo.lua](ElvUI_BounceCombo.lua)) following ElvUI's module pattern:

- **Module registration:** `E:NewModule("ComboBounce", "AceHook-3.0", "AceEvent-3.0")` — uses Ace3 mixins for hooking and events.
- **Default profile:** Defaults are stored in `P.comboBounce` (ElvUI's profile table), picked up automatically by ElvUI's db system.
- **Hook strategy:** On `PLAYER_ENTERING_WORLD`, `SecureHook` patches `ClassPower.PostUpdate` on `ElvUF_Player`. Animations are created lazily on first combo point gain per frame.
- **Animation:** Each combo point frame gets a `bounceAnim` AnimationGroup with two chained Scale animations (scaleUp → scaleDown). Settings (scale/duration) are applied at creation and refreshed via `UpdateAllSettings()` when the user changes options.
- **Options panel:** Injected into `E.Options.args.comboBounce` (AceConfig group), surfaced in ElvUI's config UI.

## Development Notes

- No build system — Lua files are loaded directly by the WoW client via the `.toc` file.
- To test: copy/symlink the addon folder into `Interface/AddOns/`, reload WoW UI with `/reload`.
- The `.toc` `## Interface:` version must match the client patch (currently `20505` for TBC Anniversary 2.5.5).
- ElvUI globals used: `E`, `L`, `V`, `P`, `G` (unpacked from `ElvUI`), `ElvUF_Player`.

---
> Source: [rbleuse/ElvUI_BounceCombo](https://github.com/rbleuse/ElvUI_BounceCombo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->

## ultimatecursor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UltimateCursor is a World of Warcraft (Retail) addon that enhances the mouse cursor with visual indicators for GCD (Global Cooldown), casting, swing timer, health, power, and other combat information. The addon displays circular rings around the cursor that fill or deplete based on game state.

**Target WoW Version:** 11.1.0 (Retail - Interface version 110100)

## Architecture

The addon follows a modular structure with a central namespace (`UltimateCursor` or `UC`):

```
Localization.lua  → Global namespace creation, localization strings (German)
Config.lua        → Default settings, SavedVariables management, slash commands
Core.lua          → Main frame, event handling, state machine, update loop
Rings.lua         → Ring rendering using line segments, extra visual features
CastBar.lua       → Cast bar functionality (ring and bar styles)
Options.lua       → Settings UI using WoW's frame templates
```

**Load order is defined in `UltimateCursor.toc` and must be maintained** - Localization first (creates global table), then Config, Core, Rings, CastBar, Options.

### Key Patterns

- **Namespace:** All modules access the shared `UltimateCursor` table (aliased as `UC`)
- **Configuration:** `UC.db` references `UltimateCursorDB` (SavedVariables), merged with `UC.defaults`
- **State:** `UC.state` tracks runtime state (combat, casting, GCD timing, cursor position)
- **Dot-notation options:** `UC:GetOption("rings.gcd.enabled")` and `UC:SetOption(path, value)`

### Ring System (Rings.lua)

Rings are drawn using WoW's Line API with 64 segments per ring. Each ring object has:
- `SetValue(value, maxValue)` - controls segment visibility based on percentage
- `SetColor(r, g, b, a)`, `SetRadius()`, `SetThickness()`, `Show()`, `Hide()`

Ring types: `middle` (static), `gcd`, `cast`, `swing`, `health`, `power`

### Event-Driven Updates

- `OnUpdate` runs every frame (throttled by `performance.updateThrottle`)
- WoW events trigger state changes via `UC:OnEvent()`
- Combat log parsing (`COMBAT_LOG_EVENT_UNFILTERED`) handles swing timer

## Slash Commands

- `/uc` or `/ultimatecursor` - Opens options panel
- `/uc toggle` - Enable/disable addon
- `/uc reset` - Reset all settings to defaults

## WoW API Considerations

- Uses `C_Spell.GetSpellCooldown` (Retail) with fallback to legacy `GetSpellCooldown`
- Frame strata: HIGH, level 100 for visibility
- Coordinates scaled by `UIParent:GetEffectiveScale()`
- Uses standard WoW UI templates: `InterfaceOptionsCheckButtonTemplate`, `OptionsSliderTemplate`, `UIDropDownMenuTemplate`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PhilippPlg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

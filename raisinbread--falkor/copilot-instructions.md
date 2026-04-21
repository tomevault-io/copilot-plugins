## falkor

> - All modules must use nested state objects: `self.moduleName = {}`


# Falkor Module Standards

## State Management
- All modules must use nested state objects: `self.moduleName = {}`
- State initialization functions must be named `init*` (e.g., `initPlayer`, `initBalance`)
- Never use flat properties on `Falkor` object for module state

## Function Naming Conventions
- `init*` - Module initialization functions
- `start*/stop*` - State change functions (enable/disable features)
- `handle*` - Event handler functions (called by triggers/balance system)
- `check*` - Validation/checking functions (return boolean or status)
- `process*` - Data processing functions (transform/parse data)
- `parse*` - Parsing functions (extract data from strings)

## Constants Management
- **Single use**: Define inline where used (no extraction needed)
- **2+ uses in same file**: Extract to module-level `local` constants at top of file
- **Cross-file use**: Place in `constants.lua` under `Falkor.PATTERNS` or similar namespace
- All magic strings/numbers must be named constants, never hardcoded

## Registry System
- All triggers and aliases must use `Falkor:registerTrigger()` and `Falkor:registerAlias()`
- Never call `tempTrigger`, `tempRegexTrigger`, or `tempAlias` directly
- Registry automatically tracks and manages all triggers/aliases

## Configuration
- All configurable values must be in `config.lua` under `Falkor.config`
- Access via `Falkor.config.module.setting` or use `Falkor:getConfig("module.setting")`
- Never hardcode configuration values in module code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raisinbread) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

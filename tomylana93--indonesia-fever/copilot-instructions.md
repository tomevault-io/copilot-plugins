## indonesia-fever

> Project: Indonesia Fever 1

Project: Indonesia Fever 1

Purpose:

- A mod for Transport Fever 2 providing Indonesian-themed content: name generators (towns, streets, persons), custom map/terrain generators (tropical rivers, trees/rocks placement), vehicle filtering for historical Indonesian context, configurable historical infrastructure years, and other localized content.

Authors & Credits:

- Creator: tomylana93
- Credits mention a signal model (DFN)

Tech stack / file types:

- Lua scripts for game/mod logic (`mod.lua`, `strings.lua`, scripts under `res/scripts/`)
- Game assets: models, materials, meshes, textures under `res/` (standard Transport Fever 2 asset layout)
- Encoding: UTF-8
- No compiled languages inside this mod; uses Transport Fever 2 mod API and engine.

Key entry files and locations:

- `mod.lua` (top-level): mod metadata, parameters, `runFn` & `postRunFn` logic
- `strings.lua`: localization strings (English shown)
- `res/scripts/vehicle_filter.lua`: vehicle filtering logic
- `res/scripts/personnameutil.lua`: name generation utilities
- `res/config/` (contains `game_script/`, `name2/`, `terrain_generators/`)
- `GEMINI.md`: development notes mentioning Gemini CLI assistance

Project structure (high-level):

- Top-level: `mod.lua`, `strings.lua`, `GEMINI.md`, `documents/`, `res/`
- `res/`: `models/`, `materials/`, `mesh/`, `textures/`, `scripts/`, `config/`

Runtime notes:

- Uses the Transport Fever 2 mod API (calls like `game.config`, `addFileFilter`, `getCurrentModId()`)
- Parameters expose configurable historical toggles and vehicle/animal filters via the mod options UI

Platform:

- Development on Windows (workspace contains VS Code tasks using `msbuild` for related staging folders).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomylana93)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomylana93)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

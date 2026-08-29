## open-wb

> Implement against [`docs/superpowers/specs/2026-08-15-tech-stack-design.md`](docs/superpowers/specs/2026-08-15-tech-stack-design.md). That file is the source of truth for architecture, `.box` format, catalog rules, v1 scope, and non-goals.

# Open WB — agent notes

Implement against [`docs/superpowers/specs/2026-08-15-tech-stack-design.md`](docs/superpowers/specs/2026-08-15-tech-stack-design.md). That file is the source of truth for architecture, `.box` format, catalog rules, v1 scope, and non-goals.

## Stack

Godot **4.7.1** standard (not .NET). GDScript only. Jolt. Forward+. MIT.

## Rules

- Worlds are JSON `.box` documents, not Godot scenes.
- The grid instantiates catalog packed scenes. Do not use `GridMap` as save format or source of truth.
- `core/` must not reference a specific tile id. New verbs are new catalog entries.
- Creators get an authored behavior catalog. No visual scripting, mods, or user shaders.
- v1 is desktop play + create + test-play of one box. Campaign, Discover, netcode, web, mobile, Rust, and C# are out.

## Layout

`catalog/` tiles · `core/box` documents · `core/grid` stamp/erase · `core/session` play/create · `boxes/` official levels · `tests/` gdUnit4

## Verify

Open the project in Godot 4.7.1 and run `res://core/session/main.tscn`. After tests exist: run gdUnit4 headless on `tests/`.

---
> Source: [koolkao/open-wb](https://github.com/koolkao/open-wb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->

## livingatlas

> Living Atlas is a desktop-first C#/.NET application for building nested RPG maps:

# AGENTS.md

## Project

Living Atlas is a desktop-first C#/.NET application for building nested RPG maps:

region -> city -> district -> building -> battle map.

The core idea is a living multi-scale atlas:
- a city contains districts;
- a district can be opened and edited as a separate map;
- the edited district returns to the city map as a scaled preview;
- later, the city can appear on a region map as a miniature of itself.

## Tech stack

- C#
- .NET 10
- Avalonia UI
- SkiaSharp
- MVVM
- xUnit

## Solution structure

- `src/LivingAtlas.Desktop` - Avalonia UI, windows, views, view models.
- `src/LivingAtlas.Domain` - pure domain model, geometry, maps, scale, hierarchy.
- `src/LivingAtlas.Editor` - editor tools, commands, selection, history.
- `src/LivingAtlas.Rendering` - SkiaSharp rendering, grid, map objects, previews, LOD.
- `src/LivingAtlas.ProjectSystem` - save/load, project manifest, JSON files.
- `src/LivingAtlas.Export` - PNG/WebP export, later Foundry VTT export.
- `src/LivingAtlas.AI` - AI provider abstractions, later structured generation.
- `tests/LivingAtlas.Tests` - xUnit tests.

## Architecture rules

- Domain layer must not reference Avalonia.
- Domain layer must not reference SkiaSharp.
- Domain layer must not use file system APIs.
- Domain layer must not use AI APIs.
- Rendering code must not own domain state.
- UI code must not contain domain rules.
- Editor commands should be testable without UI.
- Prefer small classes with clear responsibility.
- Prefer records for immutable value objects.
- Keep public APIs simple and explicit.

## Required checks

After changes, run:

```bash
dotnet build
dotnet test
```

## Current roadmap focus

Do not implement AI first.

The first milestone is:

- Create a city map.
- Create a district inside it.
- Open the district as a child map.
- Edit simple streets/buildings.
- Return to the city map.
- Show the district as a scaled preview inside the city.

## Do not implement yet

- full AI generation;
- first-person view;
- multiplayer;
- marketplace;
- full 3D;
- seamless world zoom;
- cloud sync;
- procedural world generation;
- image generation;
- Foundry export before the basic map editor works.

## Coding style

- Use nullable reference types.
- Avoid large god classes.
- Avoid hidden global state.
- Add tests for domain and editor logic.
- Do not add new dependencies without explaining why.

---
> Source: [winnietheplayer/LivingAtlas](https://github.com/winnietheplayer/LivingAtlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

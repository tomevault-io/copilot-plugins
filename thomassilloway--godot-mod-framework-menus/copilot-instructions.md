## godot-project

> Rules for updating godot project script, scene or other related files

# Overview

- Assume Godot 4.3 is being used

## Project Goals

- A godot game made with mods from the ground up
- All of the mods and the base game are in the projects folder
- The base game is called `main`
- The other projects are included in `main` by exporting them as zip files
- The code in `main` loads all of the mod zip files at startup and loads their `init_scene.tscn`

## Project Structure
- ai_coding_assistant (files that help Cursor write code)
- build (the binary files of the game when they are exported)
- documentation (relevant documentation for godot extensions)
- projects (main game project and all mod projects)
- scripts (any scripts used globally for all projects)

## Implementation details
- Ensure there's a newline at the end of each file

---
> Source: [ThomasSilloway/Godot-Mod-Framework-Menus](https://github.com/ThomasSilloway/Godot-Mod-Framework-Menus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->

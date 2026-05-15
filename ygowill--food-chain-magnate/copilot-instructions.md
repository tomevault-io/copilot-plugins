## food-chain-magnate

> - `core/`: engine-first, reusable gameplay logic (avoid UI/node dependencies). Key areas include `core/engine/`, `core/state/`, `core/actions/`, `core/rules/`, `core/map/`, and `core/tests/` (pure logic tests).

# Repository Guidelines

## Project Structure & Module Organization

- `core/`: engine-first, reusable gameplay logic (avoid UI/node dependencies). Key areas include `core/engine/`, `core/state/`, `core/actions/`, `core/rules/`, `core/map/`, and `core/tests/` (pure logic tests).
- `ui/`: Godot scenes and scripts, including `ui/scenes/tests/` (runnable test scenes) and the main menu scene `ui/scenes/main_menu.tscn`.
- `gameplay/`: gameplay validators and action wiring on top of `core/`.
- `data/`: JSON/config-driven game data (`data/maps/`, `data/employees/`, etc.).
- `assets/`: audio/fonts/images used by the UI.
- `docs/`: design/architecture/testing notes (start with `docs/testing.md`).
- `tools/`: developer scripts (notably `tools/run_headless_test.sh`).

## Build, Test, and Development Commands

This is a Godot 4.5 project (`project.godot`).

- Open editor: `godot --editor --path .`
- Run game (uses project main scene): `godot --path .`
- Run all tests headless (recommended, default timeout 120s): `tools/run_headless_test.sh res://ui/scenes/tests/all_tests.tscn AllTests`
- Run one test scene: `tools/run_headless_test.sh res://ui/scenes/tests/replay_test.tscn ReplayTest 20`

## Coding Style & Naming Conventions

- GDScript: follow existing style in nearby files; keep indentation consistent (tabs are common in this repo).
- Naming: `snake_case` for files/scenes (e.g., `produce_food_test.gd`), `PascalCase` for `class_name` types, and clear, intention-revealing function names.
- Prefer pure logic in `core/` and keep UI glue in `ui/` (tests should stay deterministic and headless-friendly).

## Testing Guidelines

- Prefer adding logic tests under `core/tests/*_test.gd`, exercised by scenes in `ui/scenes/tests/*_test.tscn`.
- Headless tests must support `-- --autorun`, print machine-greppable `START/PASS/FAIL`, and exit via `get_tree().quit(0|1)`.
- Use `tools/run_headless_test.sh` to enforce timeouts and capture logs in `.godot/*.log`.

## Commit & Pull Request Guidelines

- Git history isn’t available in this workspace; use a consistent convention like `type(scope): summary` (e.g., `fix(core): validate map baking inputs`).
- PRs: include a short problem statement, test command/output, and screenshots/GIFs for UI changes. Link relevant docs/issue IDs when applicable.

## Security & Configuration Tips

- Ensure `godot` is a real CLI on `PATH` (avoid shell aliases in scripts). See `docs/testing.md` for macOS setup notes.
- Generated/runtime folders like `.godot/` and `.tmp_home/` are used for logs and headless runs; avoid treating them as source of truth.

## Agent Behavior & Stability Requirements

The agent must behave as a conservative, correctness-first Godot engineer.

- Assume Godot 4.x APIs only.
- Never invent engine features or methods.
- If unsure, say so and suggest verification.
- Prefer explicit, maintainable solutions over clever abstractions.
- Avoid refactoring unrelated code unless explicitly requested.
- Preserve existing structure and intent when modifying files.

## Indentation & Formatting (Critical)

⚠️ Indentation errors are considered serious defects.

The agent must:

- Use tabs for indentation (matching existing files).
- Never mix tabs and spaces.
- Re-check indentation after every modification.
- Be especially careful in:
  - Nested conditionals
  - match statements
  - signal callbacks
  - loops inside _process / _physics_process

If indentation is uncertain, the agent must call it out explicitly before finalizing code.

## Conservative Development Principles

- Prefer correctness and clarity over speed.
- Do not guess engine behavior.
- Avoid speculative fixes.
- Favor small, localized changes.
- When behavior is ambiguous, ask for clarification instead of assuming.

## Pre-Change Validation Checklist

Before considering a change complete, the agent should mentally verify:

- No indentation regressions were introduced.
- Code runs headless where applicable.
- Tests remain deterministic.
- No unintended dependency on editor-only features.

---
> Source: [ygowill/food_chain_magnate](https://github.com/ygowill/food_chain_magnate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->

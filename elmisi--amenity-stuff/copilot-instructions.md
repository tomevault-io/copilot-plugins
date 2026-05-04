## amenity-stuff

> This repo is a Textual (Python) TUI that scans a source folder, extracts content (PDF / images / office / text-ish), calls a local LLM (Ollama) for structured analysis + classification, and can move files into an archive.

# Agent Guidelines (amenity-stuff)

This repo is a Textual (Python) TUI that scans a source folder, extracts content (PDF / images / office / text-ish), calls a local LLM (Ollama) for structured analysis + classification, and can move files into an archive.

These guidelines document the conventions used so far and should be followed for future development.

## General Principles
- Prefer small, focused changes; avoid incidental formatting churn.
- Fix root causes rather than layering workarounds.
- Keep the project general-purpose; avoid dataset-specific rules unless they can be expressed as user-configurable heuristics.
- Maintain an interactive TUI: never block the Textual event loop with I/O, OCR, or LLM calls.

## Code Structure
- Keep UI concerns separate from analysis logic:
  - UI rendering/formatting helpers live in dedicated `ui_*.py` modules.
  - Analysis / extraction / OCR / LLM prompt+parse live outside the UI layer.
- Prefer “data in / data out” functions for logic that doesn’t need `App` context (easier to test and refactor).
- Use dataclasses (or TypedDicts) for structured results and configuration to reduce “loose dict” usage.

## Textual / TUI Conventions
- Use workers/threads for long-running operations (scan, extraction, OCR, LLM).
- UI updates should be scheduled from the main thread (e.g. via `call_from_thread` or safe message passing), not performed directly inside worker threads.
- Prefer a single source of truth for per-file state; the table is a view of that state.
- Keyboard shortcuts should be discoverable:
  - Document them in `README.md`.
  - Keep bindings stable; when they change, update docs in the same PR/commit series.

## Refactoring Rules (Behavior-Preserving)
When refactoring, do not change functionality unless explicitly requested:
- Avoid altering prompts, heuristics, defaults, or UX flows.
- Move code in small steps:
  1) extract helper with identical logic,
  2) replace call sites,
  3) delete dead code.
- Keep function signatures stable unless there is a clear simplification.
- Ensure imports remain acyclic and modules have single, clear responsibility.

## Naming & Typing
- Use descriptive names (`analysis_config`, `extract_result`, `task_state`) over abbreviations.
- Prefer `pathlib.Path` for filesystem paths.
- Add type hints on public functions and key internal helpers; keep types pragmatic (don’t over-annotate).

## Performance & Safety
- Measure before optimizing: record timing breakdowns (extract/OCR/LLM) in results and show them in the details panel.
- Protect the user’s data:
  - Default to local-first behavior.
  - When adding providers/models, clearly document where content is sent.
- OCR is optional and may be slow; keep it configurable (mode/time budgets) and fail gracefully.

## Configuration & Cache
- Persist user configuration in `~/.config/amenity-stuff/config.json`.
- Cache analysis results per-source under `<source>/.amenity-stuff/` keyed by `(path, size, mtime)`.
- Cache invalidation should be explicit and user-driven (reset row / reset all / force reanalysis).

## Commits
- Commit logically grouped changes with clear messages (e.g. `tui: refactor settings screen`).
- Keep refactors separate from behavior changes when possible.

## Versioning (Repo / App)
- The current version is stored in `VERSION` and mirrored in `pyproject.toml`.
- Bump the **patch** version only for changes to **Python code** in `archiver/`.
- Do NOT bump the version for:
  - Documentation-only commits (README.md, AGENTS.md, CLAUDE.md, etc.)
  - Shell scripts (install.sh, uninstall.sh)
  - Configuration files
- Helper: run `python3 scripts/bump_version.py` before committing (updates both files).

## Local Installation (development)
- After bumping the version, always refresh the local install:
  ```bash
  ~/.local/share/amenity-stuff/venv/bin/pip install -e .
  ```
- This updates the system-wide `amenity-stuff` command to use your local changes.
- For quick tests without installing, run directly: `python3 -m archiver`

## SOLID Principles

This codebase follows SOLID principles to ensure maintainability and extensibility:

### Single Responsibility (SRP)
- Each module has one clear responsibility:
  - `extractors/` modules handle content extraction (one per format)
  - `prompts.py` manages all LLM prompt templates
  - `llm_backend.py` defines the LLM interface abstraction
  - `utils_parsing.py` provides text/date/amount parsing utilities
  - `utils_filename.py` handles filename manipulation
- Keep functions focused: if a function does extraction AND parsing AND LLM calls, split it.

### Open/Closed Principle (OCP)
- New LLM backends can be added without modifying existing code (implement `LLMBackend` protocol)
- New extractors can be added by creating a new module in `extractors/` and registering in `registry.py`
- Prompts are externalized in `prompts.py` for easy customization without touching logic

### Liskov Substitution (LSP)
- Any `LLMBackend` implementation must honor the protocol contract
- `OllamaBackend` and future backends must return `LLMResponse` with consistent semantics
- Backward-compatible module-level functions wrap the new classes

### Interface Segregation (ISP)
- `LLMBackend` protocol is minimal: just `generate()` method
- `BaseLLMBackend` adds optional convenience methods (`generate_with_image_file`)
- Extractors expose only what's needed: `extract_X()` functions returning `(text, meta)`

### Dependency Inversion (DIP)
- High-level modules (`analyzer.py`) depend on abstractions (`LLMBackend` protocol)
- Configuration is injected via `AnalysisConfig` dataclass
- Avoid hardcoded model names; use configuration

### Applying SOLID in Practice
- When adding a new feature, identify which module owns that responsibility
- When refactoring, extract helpers with identical logic first, then swap call sites
- Prefer composition over inheritance; use protocols for loose coupling
- Keep backward compatibility when changing interfaces

---
> Source: [elmisi/amenity-stuff](https://github.com/elmisi/amenity-stuff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->

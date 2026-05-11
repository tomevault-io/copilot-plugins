## smooveplugin

> - `Source/` contains all plugin code:

# Repository Guidelines

## Project Structure & Module Organization

- `Source/` contains all plugin code:
  - `PluginProcessor.*` (audio/DSP orchestration) and `PluginEditor.*` (UI).
  - `Source/DSP/` spectral processing and resonance suppression components.
  - `Source/GUI/` custom look-and-feel and interactive controls.
  - `Source/Parameters/` parameter IDs and APVTS layout.
- `JUCE/` is a git submodule (framework). Avoid editing it in PRs unless explicitly required.
- `Resources/` is reserved for future assets (currently empty).
- `build/` is CMake output (generated; don’t commit build artefacts).

## Build, Test, and Development Commands

Typical CMake flow (macOS/Linux):

- Configure: `cmake -S . -B build`
- Build: `cmake --build build --config Release`

Build outputs land under `build/Smoove_artefacts/` (e.g., `Release/VST3`, `Release/AU`, `Release/Standalone`). If `JUCE/` is missing, initialize submodules: `git submodule update --init --recursive`.

## Coding Style & Naming Conventions

- Language: C++17 (see `CMakeLists.txt`).
- Formatting: follow the existing style in `Source/` (4-space indent, braces on their own line, clear JUCE naming).
- Naming: classes in `PascalCase`, members in `camelCase`, files grouped by responsibility (DSP/GUI/Parameters).

## Testing Guidelines

- Automated unit tests are not set up yet.
- Minimum verification for changes:
  - `cmake --build build --config Debug` (or `Release`) succeeds.
  - Standalone app launches (`build/Smoove_artefacts/Release/Standalone/`).
  - Plugin loads and passes audio in a host (AU in Logic, VST3 in Reaper) without crashes, clicks, or parameter glitches.

## Commit & Pull Request Guidelines

- Commits: short, imperative summaries (e.g., “Add…”, “Fix…”, “Implement…”), matching existing history.
- PRs should include: a brief description, how to test (host + format), any UI screenshots/video for GUI changes, and notes on CPU/latency impact for DSP changes.

## Security & Configuration Tips

- Do not commit local plugin installs, signing/notarization files, or machine-specific IDE settings.
- Keep DAW/host-specific paths and secrets out of the repo; prefer environment variables or local-only documentation when needed.

---
> Source: [robfosterdotnet/SmoovePlugin](https://github.com/robfosterdotnet/SmoovePlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

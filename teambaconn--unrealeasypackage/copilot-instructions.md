## unrealeasypackage

> A cross-platform (Windows-first) **desktop app that gives Unreal Engine projects saveable, reusable build profiles**. Configure a build once, save it, and run it on demand - no hand-writing UAT command walls, no per-build tuning headaches. It wraps Unreal's `RunUAT BuildCookRun`, auto-detects the environment, streams live build logs, and manages the disk footprint a build leaves behind.

# UnrealEasyPackage

## Introduction
A cross-platform (Windows-first) **desktop app that gives Unreal Engine projects saveable, reusable build profiles**. Configure a build once, save it, and run it on demand - no hand-writing UAT command walls, no per-build tuning headaches. It wraps Unreal's `RunUAT BuildCookRun`, auto-detects the environment, streams live build logs, and manages the disk footprint a build leaves behind.

**Who it's for:** solo devs / small teams who find Unreal's native packaging opaque, hard to reuse, hard to tune per-build, and whose projects balloon with scattered build artifacts they can't easily track or clean.

**Problems it solves:**
1. **Opaque & non-reusable** - packaging means long, fragile UAT command lines (e.g. a hand-written `build_worker.bat`). No saved, reusable, tunable profiles.
2. **Massive scattered footprint** - a single *Development* build can produce **~38 GB of regenerable artifacts** for a ~4.8 GB shippable package (full breakdown → [docs/build-footprint.md](docs/build-footprint.md)).
3. **Inherent duplication** - `BuildCookRun` makes up to **4 copies** of game data in separate folders: source `Content/` → `Saved/Cooked/` → `Saved/StagedBuilds/` → archive dir. This is *why* projects double in size, and why cleanup is currently manual and error-prone.

## Docs
- [Requirements](docs/requirement.md) - full MVP feature spec
- [User experience](docs/user-experience.md) - screens, navigation, and the high-level user flow
- [Data storage](docs/data-storage.md) - where templates, profiles, build history, logs, and settings live (app folder vs `.uep/`, JSON vs SQLite)
- [Build commands](docs/build-commands.md) - how `RunUAT BuildCookRun` packages a game, the full parameter reference, and which params to expose in profiles vs auto-manage/hide
- [Build footprint & cleanup](docs/build-footprint.md) - how a build scatters data across your project, what's safe vs unsafe to delete, and the supporting internet findings
- [Design system](docs/design-system.md) - canonical UI color palette, surfaces, and hover/active states (Mantine + neutral Unity/Unreal theme)

## Tech stack
- **Shell:** **Tauri 2** (Rust core + OS WebView2) + **Vite** (frontend dev/HMR) + Tauri CLI bundler (NSIS/MSI installer; dmg/AppImage later). TypeScript frontend, Rust backend. The IPC boundary is `#[tauri::command]` + events; TS types are generated from Rust via `tauri-specta` (`specta` + `specta-typescript`).
- **UI:** React + **Mantine** primitives + Tabler icons. The Build Logs console and Dashboard charts are deliberately **lightweight custom components** - a structured gutter + per-line severity-tint + filter console (not xterm.js) and dependency-free SVG charts (not `@mantine/charts`) - so no extra UI deps. (xterm.js / `@mantine/charts` / `@tanstack/react-virtual` were evaluated and dropped.)
- **State:** **Redux Toolkit** (frontend UI state); **Tauri managed state** (`Mutex`/`RwLock`) for the live build run (backend).
- **Detection (Rust):** `winreg` (engine registry), `walkdir` (maps), `rust-ini` (`.ini`), `serde_json` (`.uproject`).
- **Process (Rust):** `tokio::process` (spawn + stream + clean kill). Footprint scanning via `walkdir` on a blocking thread (off the UI thread).
- **Persistence:** `serde_json` files (profiles/templates/settings) validated by `serde` + a hand-rolled `BuildConfig::validate_profile()` check; `rusqlite` (bundled) for the derived build-history index.
- **Testing:** `cargo test` on the logic that warrants it (arg builder, log classifier, tag round-trip). Pragmatic, not blanket - optimize for iteration speed.

## Architecture rule
Business logic lives in **Rust** under `src-tauri/src/` as plain modules, behind **thin `#[tauri::command]` wrappers** (`commands.rs`) - the only Tauri-facing layer. The **webview (React) is a thin UI** that calls those commands and listens for events; it holds no build logic. Keep the modules Tauri-agnostic where it's natural (inputs → data/errors) so they stay easy to reason about, reuse, and extend - a future headless CLI is then *possible* but **not** a structural mandate.

Modules under `src-tauri/src/`: `profiles/` (serde schema, load/save) · `unreal/` (engine detect, `.uproject`/`.ini` parse, BuildCookRun arg builder) · `pipeline/` (the **phase registry** - the extensibility seam; the executor, Jenkins-style graph, profile editor & per-phase timing all derive from it, so adding a phase is one registry entry) · `runner/` (DAG executor over `pipeline/`: `tokio::process` spawn per phase, log stream, warning/error parser, per-phase timing) · `footprint/` (`walkdir` scanner + categorization rules; also powers the Clean-up phase) · `history/` (build records) · `commands.rs` (`#[tauri::command]` glue) · `state.rs` (Tauri managed state).

The pipeline is **separate-process** (Jenkins-style): Build / Cook / Stage+Pak+Archive run as child processes, **Copy Extras / Clean-up** as app-owned tasks; phases come from `pipeline/`'s registry so the set is **extensible** without touching the executor or UI. See [docs/build-commands.md](docs/build-commands.md#8-phase-decomposition-separate-processes-parallelism-and-timing).

Beyond projects, the app also **packages plugins** ([docs/requirement.md](docs/requirement.md) R7): the gate opens a `.uplugin` as well as a `.uproject` (recents tag each PROJECT/PLUGIN), and a plugin lands on a single **Tools** tab whose **Package Plugin** action runs `RunUAT BuildPlugin … -rocket` against a user-picked engine (standalone - **no host project**; see [docs/build-commands.md](docs/build-commands.md#9-plugin-packaging-runuat-buildplugin)). It reuses the same modules: `unreal/` (`uplugin.rs` parse + `engine::enumerate_registry_engines`), `unreal::args` (the `BuildPlugin` command builder), and the `runner/` substrate (`spawn_plugin_package` - same live stream + cancel, but no history record). Per-plugin machine-local settings (remembered engines + output folder + folder name) live in one plain JSON beside the `.uplugin` - `<plugin_root>/.uap/settings.json` (its own git-ignored `.uap/` folder, separate from the engine-side `.uep/`), never in the host project root.

## Privacy & public-repo rules
This repo is **public**. Never commit anything personal or machine-specific.

- **No personal/machine data in tracked files** - no absolute machine paths, real project names, engine-association GUIDs, emails, usernames, or host identifiers. Use neutral placeholders instead (`SampleProject`, `C:\Projects\SampleProject`, a clearly-fake GUID).
- **Real values live only in [CLAUDE.local.md](CLAUDE.local.md)** (gitignored). It holds the reference project/engine paths, the real engine GUID, and the placeholder→real mapping - for the AI to use locally without exposing anything.
- **Tests must be hermetic** - pure logic over `tempdir`/in-memory/string fixtures. If a check can only pass by touching *this* machine (reading the registry, real engine/project paths, deleting real dirs), **do not** write it as an automated test. Remove it and document it as a **human-run** manual verification; the human runs it and reports the result.

---
> Source: [TeamBaconn/UnrealEasyPackage](https://github.com/TeamBaconn/UnrealEasyPackage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->

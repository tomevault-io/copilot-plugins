## accessforge

> A 1-click accessibility mod platform for blind gamers. Discovers, installs, and updates screen reader mods for mainstream games.

# AccessForge

A 1-click accessibility mod platform for blind gamers. Discovers, installs, and updates screen reader mods for mainstream games.

## User

- Blind, screen reader user
- Uncertainties: ask briefly, then act
- Output: NO `|` tables, use lists

## Project Info

- **Language:** Rust (edition 2024)
- **GUI framework:** wxDragon (wxWidgets bindings for Rust)
- **HTTP client:** ureq (sync, no async runtime)
- **Error handling:** anyhow
- **Serialization:** serde + serde_yaml (manifests) + serde_json (state file)
- **GitHub org:** AccessForge
- **Application name (in GUI):** AccessForge
- **Application name (executable):** AccessForge

## Prerequisites

- Rust (latest stable via rustup)
- CMake
- LLVM (for bindgen, set `LIBCLANG_PATH=C:\Program Files\LLVM\bin`)
- Visual Studio 2022 Build Tools with C++ workload + Windows 11 SDK
- Screen reader (NVDA) for accessibility testing

## Environment

- **Portable app:** single exe. State file at `%LOCALAPPDATA%/AccessForge/state.json`. No installer, no registry entries.

## Architecture

- `src/main.rs` — entry point, CLI arg parsing
- `src/ui/` — wxDragon GUI code
- `src/cli/` — CLI commands (init, install, package)
- `src/manifest/` — YAML manifest parsing and validation
- `src/state/` — local state file (state.json) read/write
- `src/installer/` — mod/loader/dependency installation logic
- `src/registry/` — GitHub topic-based mod discovery
- `src/steam/` — Steam library detection (libraryfolders.vdf parsing)

See if there is a crate that can help the steam library detection.
Take extra care to separate UI from bussness logic, so if we want to switch GUI framework later on, we can.


## Key Design Decisions

- **Manifest file:** `accessforge.yml` in mod repo roots (YAML format)
- **State file:** `%LOCALAPPDATA%/AccessForge/state.json` (JSON, machine-generated)
- **Mod discovery:** GitHub topic enumeration (repos tagged with a specific topic)
- **No central registry repo:** mods are discovered via GitHub topics
- **Dependency mirrors:** non-GitHub dependencies (e.g. Nexus-only tools) mirrored in separate GitHub repos under the AccessForge org
- **Pinned dependency versions:** mod author controls when dependency updates go out, no auto-upgrading
- **Loader knowledge baked in:** platform knows how to install ue4ss, bepinex, melonloader and where each puts mods
- **Game path detection:** Steam library scanning via store IDs, fallback to asking user once

## Specs

Read these before implementing:
- `manifest-spec.md` — YAML manifest format, field reference, validation rules
- `state-file-spec.md` — local state JSON schema, write safety, recovery
- `design-notes.md` — platform philosophy, loader support, developer workflow, known mods

## Coding Rules

- Logs/comments: English
- Private fields: `_snake_case` (Rust convention: no underscore prefix, just snake_case)
- Error handling: `anyhow::Result` for application code, `thiserror` for library errors
- No `unwrap()` in production code — use `?` or `.context("message")?`
- Tests: `cargo test`, use `rstest` for parameterized tests if needed

## Coding Principles

- **Accessible first** — every UI element must work with screen readers. Test with NVDA.With WX the GUI should basically be accessible out of the box.
- **Portable** — single exe, state in `%LOCALAPPDATA%/AccessForge/`. No installer, no registry entries.
- **Minimal footprint** — release builds: `strip = true`, `opt-level = "z"`, `lto = true`, `panic = "abort"`, `codegen-units = 1`
- **Simple over clever** — sync HTTP (ureq), no async runtime, no dependency injection frameworks
- **Offline-safe** — graceful handling when GitHub is unreachable. Show cached state, don't crash.

## Release Build

```toml
[profile.release]
strip = true
opt-level = "z"
lto = true
panic = "abort"
codegen-units = 1
```

## Releasing

**Never do this automatically. Only on explicit user request.**

1. Commit all changes and push to GitHub
2. Trigger the release workflow: `gh workflow run release.yml -f version=<VERSION>`
3. Wait for the job to finish: `gh run watch`
4. Pull the version bump commit: `git pull`

## CLI Commands

```
accessforge init [--from <path>]           # create accessforge.yml manifest
accessforge install [--from <path|url>]    # install from local folder or HTTP URL
accessforge package [--from <path>]        # build release zip
accessforge setup-path                     # add AccessForge to PATH
```

## Progress Tracking

- Read `progress.md` at the start of each session to understand what's been built
- Update `progress.md` when a feature is completed or a new feature is suggested
- Move items from "Not Yet Implemented" to "Completed Features" when done
- Add new ideas/suggestions to "Not Yet Implemented" or "Open Questions"

## Session & Context Management

- ~30+ messages or ~70%+ context → remind about fresh conversation
- Before ending → update design docs and progress.md if decisions were made
- Problem persists after 3 attempts → stop, explain, suggest alternatives, ask user

---
> Source: [AccessForge/AccessForge](https://github.com/AccessForge/AccessForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->

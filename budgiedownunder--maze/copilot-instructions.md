## maze

> If this project gains a new top-level `src/` subdirectory, a new Rust crate, a new C# solution file, or a new test crate with special invocation requirements, update this file to reflect the change.

# CLAUDE.md

## Maintenance instruction

If this project gains a new top-level `src/` subdirectory, a new Rust crate, a new C# solution file, or a new test crate with special invocation requirements, update this file to reflect the change.

---

## Project overview

Experimental multi-language project exploring **Rust / C# / .NET 10 / WebAssembly interop**.

- **Domain:** maze generation and solving
- **Platforms:** Windows, Android, iOS (MAUI app), Web (REST API), Browser (WASM/JS)
- **Status:** experimental — not production

---

## Repository structure

```
src/rust/            # Rust workspace — all crates; Cargo.toml at workspace root
src/csharp/          # .NET 10 libraries, MAUI app, tests; solution files at root of this dir
src/react/           # React frontends; each subdirectory is a standalone npm project
src/docfx/           # Combined documentation generator (DocFX + cargo doc + wasm-pack doc)
docs/diagrams/       # PlantUML architecture source files
.github/workflows/   # GitHub Actions CI
```

---

## Rust workspace conventions

- All crates are declared as workspace members in `src/rust/Cargo.toml` — check there for the current list.
- Common feature flags used across the workspace:
  - `generation` — maze generation algorithm (maze crate)
  - `wasm-bindgen` — JS-targeted WASM build
  - `wasm-lite` — Wasmtime/.NET-targeted WASM build (no JS runtime)
- `maze_wasm` has **two distinct WASM build targets**: JS (via wasm-pack) and Wasmtime/.NET (via cargo direct).
- **Always read a crate's own README before running its tests** — some crates require special flags (e.g. `storage`, `maze_c`, `maze_console` require `-- --test-threads=1` due to shared file system state).

For build and test commands, see **`src/rust/README.md`** and each crate's own `README.md`.

---

## C# conventions

- Two solution files at `src/csharp/`:
  - `Build-Windows.sln` — includes mobile targets (Android, iOS)
  - `Build-Non-Windows.sln` — desktop/server only
- New projects follow the `Maze.*` naming convention. Check `src/csharp/` for the current project list.
- **Interop chain:** `maze_wasm` (Rust) → `Maze.Interop` (WASM runtime bridge) → `Maze.Api` (clean .NET wrapper) → `Maze.Maui.App`
- Runtime selection inside `Maze.Interop`:
  - Wasmtime — Windows, Android
  - Wasmer — Android emulator, iOS simulator
  - Native C via P/Invoke — iOS physical device

For build and test commands, see **`src/csharp/README.md`**.

---

## Build and test

**Always read the relevant README before building or testing.** Commands are subject to change and the READMEs are the maintained source of truth:

| Component | README |
|:----------|:-------|
| Rust workspace | `src/rust/README.md` |
| Individual Rust crates | each crate's own `README.md` |
| C# / .NET | `src/csharp/README.md` |
| Documentation | `src/docfx/README.md` |
| Web server config & auth | `src/rust/maze_web_server/README.md` |
| React web frontend | `src/react/maze_web_server/README.md` |

Do not ask for confirmation before running tests.

---

## Documentation

Build and serve instructions are in `src/docfx/README.md`. The documentation is auto-deployed to GitHub Pages on every push to `main`.

---

## CI/CD

| Workflow file | Purpose | Runs on |
|:---|:---|:---|
| `build-and-test-components-multi-os.yml` | Full build + test | Ubuntu, macOS, Windows |
| `build-and-deploy-to-github-pages.yml` | Documentation deploy | Windows only |
| `generate-png-from-puml.yml` | Auto-generate PNGs from `docs/diagrams/*.puml` | — |

---

## MAUI platform gotchas

These are runtime-confirmed bugs and constraints not derivable from reading the code:

**SwipeView transition mode**
- Android: must use `android:SwipeView.SwipeTransitionMode="Drag"` — `Reveal` renders swipe items *behind* the row content, causing a visible overlap bug.
- iOS: use `ios:SwipeView.SwipeTransitionMode="Reveal"` (native iOS swipe style, works correctly).

**SwipeItemView icon tinting**
- `SwipeItem` (platform native) inherits icon tint automatically.
- `SwipeItemView` (custom view) does not — add `toolkit:IconTintColorBehavior TintColor="White"` explicitly inside each icon `Image`.

**Shell/native nav desync**
- Blank page and broken back button after navigation is a **debug-mode artifact only**. Do not add workarounds (e.g. `PopToRootAsync`, back button overrides) in production code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/budgiedownunder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->

## py4gw-reforged-native

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`Py4GW_Reforged` is a Windows-only, **32-bit** injected DLL for Guild Wars. It is a rework of the legacy `Py4GW` library, focused on a clean C++/Python interface and supporting a proxy library on top. Inside the game process it:

- spawns a runtime thread from `DllMain` (work is kept out of `DllMain` itself),
- hooks the Guild Wars Direct3D 9 render/reset pipeline via MinHook,
- hosts an embedded CPython interpreter through pybind11,
- renders a Dear ImGui overlay and runs user Python scripts.

The build output `Py4GW.dll` is written to the **repo root** (not a `bin/` dir) so it sits beside the `fonts/`, `scripts/`, and `offsets/` payload directories it loads at runtime.

## Build

32-bit is mandatory — CMake hard-fails on non-Windows or non-Win32 architecture.

```powershell
# Preset-based (writes to ./vs2022-win32)
cmake --preset vs2022-win32
cmake --build --preset vs2022-win32-relwithdebinfo   # or vs2022-win32-debug

# Or plain
cmake -S . -B build -A Win32
cmake --build build --config RelWithDebInfo
```

The DLL is emitted directly into the repo root, next to the `fonts/`, `scripts/`, and `offsets/` payload directories it loads at runtime (`Patterns::Initialize()` defaults to `<module_dir>/offsets`), so no copy step is needed. There are no automated tests; validation is "it builds + a forced startup/shutdown path works" (see migration docs).

Source files are globbed (`GLOB_RECURSE` with `CONFIGURE_DEPENDS`) from `src/**/*.cpp`, so **adding a new `.cpp` requires re-running CMake configure**. ImGui addon sources (`src/imgui/addons/`) are compiled into a separate `imgui_addons` static lib, not the main target. `base/error_handling.h` is force-included (`/FI`) into every translation unit of `Py4GW` and `imgui_addons`.

## Runtime lifecycle

- C API in `include/Py4GW.h`: `Py4GW_Initialize()`, `Py4GW_Shutdown()`, `Py4GW_RequestShutdown()`; runtime thread `py4gw::RuntimeThread`.
- `src/Py4GW.cpp` owns the coarse state machine (mutex + `g_running` / `g_shutdown_requested` booleans). Bootstrap order: logger → `Scanner` → `Patterns` → Python → GW layer → crash handler → register ImGui + render/reset callbacks.
- `src/GW/GuildWars.cpp` drives the GW subsystem layer: `GW::Initialize()` runs an ordered `kInitSteps` table of per-module `Initialize`/`Shutdown` pairs, then enables the `MemoryPatcher` hooks. Shutdown disables patches and tears down steps in reverse. **Add a migrated module by inserting it into `kInitSteps` here** (and update the array size). Each step stamps `CrashHandler::SetContext(...)` for crash attribution.
- Render callbacks `DrawLoop`/`OnReset` only run while `g_running`; the update loop calls the Python update path every ~10 ms.

Shutdown is a strict state transition, not best-effort: mark intent → stop new work → disable hooks/restore patches → drain in-flight detours → remove hooks → delete sync primitives. Never delete a critical section or null a trampoline while a detour may still enter it; a short sleep is not a correctness boundary. Crash capture is torn down last.

## GW module anatomy

Each Guild Wars subsystem lives under `include/GW/<module>/` + `src/GW/<module>/` in namespace `GW::<module>`. Observed file split:

- `<module>.h` — all module-owned declarations: lifecycle, public callable surface, structs/typedefs/aliases, globals, and **named ownership of every resolved symbol** (functions, offsets, anchors, pointers, callsites, patch sites).
- `<module>.cpp` — discovery/setup, hook install/trampolines, lifecycle orchestration, private helpers.
- `<module>_methods.cpp` — bodies of public callable accessors/operations (omit if the module is lifecycle-only).
- `<module>_patterns.cpp` — `Resolve*` functions that call `PY4GW::Patterns::Resolve("<ns>.<name>", &g_ptr)`.
- `<module>_bindings.cpp` — pybind11 module (see below).

Shared GW data (entities, context layouts, helpers on GW types — anything without its own lifecycle) goes in `GW/context/`, **not** inside whichever manager first needed it. Shared protocol/packet/opcode declarations go in `GW/common/`. Project-wide infrastructure with no GW specificity goes in `base/`.

## Pattern / scanner system

Addresses are **not** hardcoded — they are resolved at runtime from `offsets/<module>.json`. Each file declares a `namespace`, raw `patterns` (byte `pattern`+`mask`+`offset`+`section`, or assertion anchors via `assertion_file`/`assertion_message`), and step-based `resolvers` (ops like `scan`, `dereference`, `to_function_start`, `function_from_near_call`, `add`, `validate_section`). Code resolves a final symbol with:

```cpp
PY4GW::Patterns::Resolve("agent.change_target_func", &g_change_target_func);
```

Move byte patterns, masks, sections, offsets, assertion files/messages into the JSON — do not leave them as string literals in code. The `.cpp` may execute the scan, but the module **header** must declare which symbols the module owns. Validate resolved addresses before installing hooks.

## Python bindings

The interpreter is embedded once (`src/base/python_runtime.cpp`). Bindings are split into separate embedded modules, each declared with `PYBIND11_EMBEDDED_MODULE(Py<Name>, m)` in a `<module>_bindings.cpp` (e.g. `PyAgent`, `PyMap`, `PyImGui`). **Every binding module must be explicitly `import`ed in `python_runtime::Initialize()`** — registering the macro alone does not load it. The core `Py4GW` module (logging `Console`, `SharedMemory`, minimal `ImGui`, script control) is defined inline in `python_runtime.cpp`.

User scripts loaded from `scripts/` are exec'd into a fresh module and driven by whichever of `main()` / `update()` / `draw()` they define (`draw()` runs in the render path, `update()` in the loop). Script state is `Running`/`Paused`/`Stopped`; the GIL is released after init and re-acquired around each call. `scripts/PyImGui.pyi` and `scripts/pyimgui.py` are Python-facing stubs/helpers kept in sync with the C++ bindings.

### PyImGui (ImGui control platform)

ImGui (1.92.x docking) is driven almost entirely from Python; C++ owns only the install + per-frame `NewFrame`/`Render` boundary. The single `PyImGui` module is **assembled from per-domain registrars**: `src/imgui/imgui_bindings.cpp` declares `PYBIND11_EMBEDDED_MODULE(PyImGui)` (core widgets/windows/tables/etc. inline) and at the end calls `register_*` functions declared in `include/imgui/bindings.h` and defined under `src/imgui/bindings/` — `types` (Vec2/Vec4/`color`), `enums` (uniform flag macro w/ `__or__`/`__and__`/`__xor__`/`__invert__`), `style` (live `get_style` + `StyleConfig`), `docking` (+ DockBuilder + `Dir`), `drawlist` (`ImDrawList` class via `get_window/foreground/background_draw_list`), `io` (live guarded `get_io`), `addons`. Registration order matters: `register_types`/`register_enums` run first. Host-owned APIs (context/frame lifecycle, IO event injection, font-atlas build, platform/viewport) are deliberately **not** bound. Addons are submodules `PyImGui.{filebrowser,hotkey,markdown,memory_editor,anim}`; `markdown`/`hotkey` route through bridges in their `src/imgui/addons/*_demo.cpp` TUs because `imgui_markdown.h` (non-inline) and `imHotKey.h` (needs legacy key shims) can't be included in the bindings TU. `scripts/pyimgui.py` is the ergonomic facade (scopes, `State`, `__getattr__` fallthrough); regenerate `scripts/PyImGui.pyi` in-game via `scripts/gen_pyimgui_stubs.py`; `scripts/audit_imgui_bindings.py` reports script-facing coverage.

## Conventions (enforced)

- Every project-owned header includes `base/error_handling.h` immediately after `#pragma once` (`third_party/` is exempt).
- Fatal invariants use the panic macros from `base/panic.h`: `PY4GW_ASSERT`, `PY4GW_ASSERT_MSG`, `PY4GW_REQUIRE`, `PY4GW_PANIC`, `PY4GW_UNREACHABLE`. Use the logger (`Logger::Instance()`, or the `Console` Python bindings) for diagnostics; a failure that must stop the process is not a logger-only event.
- Wrap module lifecycle, resolution, hook/patch, and callback steps with `CrashHandler::SetContext(...)` / `CrashContextScope` so the crash sidecar names the active subsystem.
- Namespaces: project infra is `PY4GW::...`; GW subsystems are `GW::<module>`; shared GW data is `GW::Context` / `GW::common`.
- Docs are ASCII-only; treat mojibake as a defect.

## Migration discipline

Much active work is porting legacy `Py4GW` (GWCA) subsystems into this tree. This is **parity migration**, governed strictly by `docs/module-migration-guide.md` and `docs/12-project-style-guide.md` — read both before any structural migration edit. Key rules:

- Refresh context first (reread the migration guide + style guide, inspect the closest migrated sibling and current `GW/context/` layout) **in the same turn** before editing structure.
- Port native managers first; do **not** auto-add Python bindings, legacy wrapper classes, or convenience layers unless asked.
- `GWCA/Managers/...` → `GW/<module>/` managers; `GWCA/GameEntities/...` → `GW/context/`; packets/opcodes → `GW/common/`; multi-consumer utilities (e.g. `MemoryPatcher`) → shared `base/` infrastructure with top-level lifecycle wiring.
- No shims for missing dependencies — migrate the prerequisite first, or stop and surface it.
- Add no new behavior/renames during a parity port; if a deviation is required, document it explicitly. Don't leave the repo half-migrated.

## Further docs

`docs/` is the authoritative deep reference (numbered `01`–`12` plus migration/parity maps and `uimgr-migration-*`). Start with `docs/README.md`. `pattern_parity_audit.md` and `docs/parity-report-*.md` track migration status against the legacy source.

---
> Source: [apoguita/Py4GW_Reforged_Native](https://github.com/apoguita/Py4GW_Reforged_Native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->

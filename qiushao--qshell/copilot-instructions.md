## qshell

> - Core application code is under `src/`:

# Repository Guidelines

## Project Structure & Module Organization
- Core application code is under `src/`:
  - `src/ui/` for Qt windows, dialogs, session tree, and terminal widgets.
  - `src/core/` for shared utilities (for example config and crypto helpers).
  - `src/scriptengine/` for Lua automation integration.
  - `src/resources/` for icons, desktop metadata, and default settings.
- Build configuration lives in root `CMakeLists.txt` plus `src/CMakeLists.txt`.
- Packaging and helper scripts are in `scripts/` (`linux-build.sh`, `macos-build.sh`, `windows-build.ps1`, `format.sh`).
- Reference docs and screenshots are in `docs/`.
- Vendored dependencies are isolated in `third_party/`; avoid editing them unless you are intentionally updating a dependency.

## Build, Test, and Development Commands
- Linux build/package: `./scripts/linux-build.sh [version]`.
- Generic local build: `cmake -B build -S . && cmake --build build`.
- Install locally (Linux/macOS): `cmake --install build`.
- macOS package: `cd build && cpack -G DragNDrop`.
- Windows build/package (PowerShell): `.\scripts\windows-build.ps1 -BuildOnly` or `.\scripts\windows-build.ps1 -Version 1.2.3`.
- Format all source and CMake files: `./scripts/format.sh`.

## Coding Style & Naming Conventions
- C++ standard: C++17 (`CMAKE_CXX_STANDARD 17`).
- Formatting is enforced by `.clang-format` (4-space indent, no tabs, LLVM-based style).
- Static analysis uses `.clang-tidy`; on Linux, CMake enables it by default (`QSLOG_CLANG_TIDY_ENABLE=ON`).
- Naming follows configured tidy rules: classes `CamelCase`, functions/variables `camelBack`, private/protected members end with `_`.

## Testing Guidelines
- There is currently no dedicated unit-test target (`ctest`/`add_test` not configured).
- Validate changes with a clean build on your target platform and functional UI checks.
- For script-engine changes, run representative Lua scripts from `scripts/lua/` (for example `startup_args_test.lua`, `timer_test.lua`).

## Commit & Pull Request Guidelines
- Follow the existing history style: short, focused, imperative summaries (commonly Chinese), e.g. `修复会话切换崩溃` or `添加启动脚本参数支持`.
- Keep each commit scoped to one logical change.
- PRs should include:
  - What changed and why.
  - Platforms verified (Linux/macOS/Windows) and commands run.
  - Screenshots or recordings for UI-visible changes.
  - Linked issue/task when applicable.

---
> Source: [qiushao/qshell](https://github.com/qiushao/qshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->

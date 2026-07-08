## pluginbased

> * @Date: 2026-05-16 21:53:22

<!--
 * @Author: zs
 * @Date: 2026-05-16 21:53:22
 * @LastEditors: zs
 * @LastEditTime: 2026-05-16 22:31:04
 * @FilePath: /PluginBased/AGENTS.md
 * @Description: 
 * 
 * Copyright (c) 2026 by zs, All Rights Reserved. 
-->
# Repository Guidelines

## Project Structure & Module Organization

This is a Qt 6 / QML generic plugin host built with CMake and C++17. The main application lives in `app/`, with QML screens in `app/qml/`. Shared runtime logic is split into `core/` for plugin management and app services, `logger/` for the spdlog wrapper, and `plugin/` for the `IAppPlugin` interface. Plugin implementations are under `plugins/`: `DummyPlugin/` is the minimal reference and `PlayPlugin/` contains FFmpeg playback code, QML views, and shaders. Packaging and deployment helpers live in `tools/`, with `package.sh` as the top-level packaging entry point.

## Build, Test, and Development Commands

Configure a development build:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake
```

Build all app, library, and plugin targets:

```bash
cmake --build build --parallel
```

Run the debug app:

```bash
./build/app/PluginBasedApp
```

Create release artifacts after a release build:

```bash
./package.sh --skip-build
```

Validate packaged binary dependencies:

```bash
python3 tools/verify.py --stage-dir /path/to/staging
```

## Coding Style & Naming Conventions

Use C++17, Qt idioms, and the existing brace style: function braces on their own line, 4-space indentation, and pointer/reference markers attached to the type where already used, for example `QObject* parent`. Name classes in `PascalCase`, methods and variables in `camelCase`, and member variables with `m_` prefixes. Keep QML component filenames in `PascalCase.qml`. Prefer CMake targets and Qt registration patterns already used in nearby modules.

## Testing Guidelines

No unit test framework or CTest targets are currently configured. For changes, at minimum run `cmake --build build --parallel` and manually launch `./build/app/PluginBasedApp`. For plugin or packaging changes, also run `package.sh` and `tools/verify.py` against the generated staging directory. Place future automated tests in a dedicated `tests/` tree and wire them through CTest.

## Commit & Pull Request Guidelines

Recent commits use short Chinese prefixes such as `[功能新增]`, `[功能修改]`, `[测试]`, and `[组件测试]`. Keep subjects imperative and specific, for example `[功能修改] 修复播放列表切换状态`. Pull requests should describe the user-visible change, list build or manual verification performed, link related issues, and include screenshots or short recordings for QML UI changes.

## Agent-Specific Instructions

Do not edit generated output under `build/` or packaged archives directly. Make source changes in `app/`, `core/`, `logger/`, `plugin/`, `plugins/`, or `tools/`, then rebuild or repackage.

---
> Source: [Zacus/PluginBased](https://github.com/Zacus/PluginBased) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

## x64dbg-automate

> x64dbg plugin that exposes a ZMQ-based automation server for remote control of the debugger.

# x64dbg-automate

x64dbg plugin that exposes a ZMQ-based automation server for remote control of the debugger.

## Build

**You must use the VS2022 cmake.** The system PATH does not have cmake. Use:

```bash
"/c/Program Files/Microsoft Visual Studio/2022/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/bin/cmake.exe" --build build64 --config Release
```

If you need to regenerate the build system:

```bash
"/c/Program Files/Microsoft Visual Studio/2022/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/bin/cmake.exe" -B build64 -G "Visual Studio 17 2022" -A x64
```

Output: `build64/Release/x64dbg-automate.dp64`

## Project structure

- `src/plugin.cpp` — Plugin callbacks, menu UI, event publishing
- `src/xauto_server.cpp` / `.h` — ZMQ server, session management, command handling
- `src/pluginmain.cpp` / `.h` — Plugin entry points and SDK boilerplate
- x64dbg plugin SDK is fetched via CMake into `build64/_deps/x64dbg-src/pluginsdk/`

## Settings

Stored in x64dbg.ini under `[XAutomate]`. Key settings:
- `Mode` — `"local"` (default) or `"remote"`
- `BindAddress` — Bind address for remote mode (e.g. `0.0.0.0`)
- `ReqRepPort` / `PubSubPort` — Fixed ports for remote mode (0 = random, local only)

---
> Source: [dariushoule/x64dbg-automate](https://github.com/dariushoule/x64dbg-automate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

## reversal-suite-mcp

> If you are a Claude Code (or other coding-agent) session with write access to this repository, read this before making changes. This file is the source of truth for how work happens here.

# Instructions for AI agents working on this repo

If you are a Claude Code (or other coding-agent) session with write access to this repository, read this before making changes. This file is the source of truth for how work happens here.

## What this project is

A native C++ MCP (Model Context Protocol) server that gives AI agents reverse-engineering superpowers on Windows: pluggable disassembly (iced-x86), pluggable emulation (Unicorn / Sogen / others), full IDA Pro / Hex-Rays integration through a persistent idalib worker, PDB pipeline with symbol-server download, PE parsing, string / xref / call-graph analysis, and a Unicorn-based decryptor scanner. One server process serves many concurrent Claude Code sessions through an in-process broker thread; each session is isolated.

## Safety rules — hard invariants

These are **non-negotiable**. Violations should fail the build.

1. **Never execute foreign binaries** from an agent tool. Emulation runs against user-supplied paths, in-process, sandboxed.
2. **Never invent a new `safety_class`.** The registry only knows about the classes declared in `src/core/safety.hpp`. Adding one requires a human review.
3. **Never write to `HKLM`, install a Windows service, or spawn `sc.exe start`** unless the user asked for it in the same session and the `--allow-system-state` flag is set.
4. **Never call `LoadLibrary` on a DLL path that came from an MCP client.** Static config only. The plugin loader is an exception: it loads DLLs only from an operator-controlled directory (`--plugins-dir`), never from a caller-supplied path, and only when `--allow-plugins` is set. See `src/plugins/loader.cpp` and `SAFETY.md`.
5. **Anything requiring elevated system access lives in operator-installed plugin addons, not in this repo's built-in tool set.** The plugin loader auto-caps at `local_probe`; `system_state` needs `--allow-plugin-elevated` AND a Windows token with admin rights (the `rsm_request_elevated_restart` builtin surfaces a UAC prompt for the operator when a plugin needs it).

## Build/test/run

- `scripts\build.ps1` — one-shot Windows build via CMake + MSBuild (Ninja fallback).
- `scripts\launch.py` — auto-launcher; ensures the server is built and running, then bridges stdio to the broker.
- `ctest --test-dir build` — runs the GoogleTest suite.
- CI: `.github/workflows/ci-windows.yml` builds on windows-latest with MSVC.

## Style

- C++23 (raised from C++20 when the iced-cpp backend landed; parts of the codebase still lean on C++20 features and that's fine). `nlohmann::json` for JSON. `spdlog` for logging.
- `snake_case` for functions and variables, `PascalCase` for types is acceptable but the existing header style is snake_case with `_t` suffix — match that.
- One class per file where reasonable.
- No exceptions across the MCP boundary — return `result_t<T>`.
- Header-only for tiny utility types; `.hpp`/`.cpp` split for anything with state or heavy includes.

## Commit rules

- No `Co-Authored-By` lines. No `Claude-Session:` links. Commits look like a human wrote them.
- Use `git config user.email` set to the private GitHub noreply email (never the operator's personal email).
- One logical change per commit. Squash noisy WIP commits before merging.

## When you add a new backend

- Implement the header interface in `src/backends/<kind>/<name>_backend.cpp`.
- Register it in the `backend_factory` for its kind (do not touch tool code).
- Add a `docs/BACKEND_<KIND>.md` entry describing config knobs, probe behavior, and safety class.
- Add a smoke test that spins the backend up in a mock mode.

## When you add a new tool

Follow `docs/ADDING_TOOLS.md`. Short version: pick a safety class, write the JSON schema in the tool file itself, register in `tool_registry`, add a test that runs the schema against a rejection case.

## When you touch the plugin loader

- The public ABI is `include/rsm/plugin.h`. It is **extern-C, additive-only** for the whole v1.x line. Renaming, reordering, or changing the type of any existing field in `rsm_plugin_manifest`, `rsm_host_api_v1`, `rsm_tool_desc`, or `rsm_tool_result` is a major-version break — don't do it without a `v2` header.
- New host callbacks go **after** the existing ones, replacing a `_reserved` slot or appending. Bump `RSM_PLUGIN_ABI_MINOR` and document the addition in the ABI stability table of `rsm-plugin-example/README.md`.
- Never remove the tool-name prefix check (`plugin_<name>_*`) or the safety cap — plugin authors depend on those being enforced.
- The loader's real bug in memory is easy to reintroduce: any host-facing state that a plugin can read after `rsm_plugin_init_v1` returns MUST have static storage duration. See `get_host_api()` in `src/plugins/loader.cpp` — this used to be a stack local and it corrupted every plugin call.
- If you change the elevation-handoff wire (`--elevated-handoff` / `--parent-pid` / `--opt-in-parent-pid`), rebuild both the parent's argv-reproduction (`build_elevated_args` in `elevation.cpp`) AND the child's config parse (`config.cpp`) in the same commit.
- Reference plugin lives at [github.com/pinefin/rsm-plugin-example](https://github.com/pinefin/rsm-plugin-example) — keep it building against the current header when you change the ABI.

---
> Source: [pinefin/reversal-suite-mcp](https://github.com/pinefin/reversal-suite-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->

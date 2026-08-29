## aidaprivate

> This repository is AiDA: a Windows reverse-engineering toolkit with an IDA Pro plugin, a standalone ImGui/DX11 IDE, and a kernel driver (WhosWho) for functional RE features. Read this file before making changes.

# AiDA Agent Instructions

This repository is AiDA: a Windows reverse-engineering toolkit with an IDA Pro plugin, a standalone ImGui/DX11 IDE, and a kernel driver (WhosWho) for functional RE features. Read this file before making changes.

**AiDA is a personal-only tool. There is NO license system, NO server, NO ARC, NO anti-tamper, NO protector, NO Sentinel driver, NO string obfuscation — all of that was permanently deleted in August 2026. Do not reintroduce any protection, obfuscation, virtualization, packing, licensing, activation, phone-home, or self-defense mechanism. AiDAStandalone.exe must boot straight into the IDE with no key prompt and no network requirement.**

## Build System

AiDA uses CMake 3.25+ with Ninja generator and MSVC on Windows. C++17, C, and ASM_MASM (kernel_symbols.cpp alone compiles as C++20).

- **BUILDS:** The user runs builds themselves unless they explicitly tell the host AI to build. When told to build: incremental: `.\build-host.cmd`. Full clean rebuild including drivers: `.\build-host.cmd -FullClean`. Driver rebuild only: `.\build-host.cmd -Drivers -CleanDrivers`. Dry-run plan: `.\build-host.cmd -PlanOnly -FullClean`.
- The wrapper writes timestamped logs under `%TEMP%\aida-build-*`, stable logs at `%TEMP%\aida_driver_build_out.txt`, `%TEMP%\aida_configure_out.txt`, `%TEMP%\aida_build_out.txt`, `%TEMP%\aida_build_verify_out.txt`, and a machine-readable summary at `%TEMP%\aida_build_summary.json`.
- The only supported configure/build preset is `ninja-msvc-release` in `CMakePresets.json`.
- Driver embed pipeline: `driver\WhosWho\WhosWho.sln` (contains WhosWho + WindMapper) builds `build-ninja\Release\WhosWho.sys`; the CMake `encrypt_drivers` target then runs `python src/encrypt_whoswho.py --from-binary build-ninja/Release/WhosWho.sys` to generate the PLAIN (unencrypted) embedded byte array in `src/whoswho_embedded.h`, which `src/driver_loader.cpp` includes. The driver binary must exist before configure/build, or `driver_loader.cpp` fails on the missing header.

## Subagent Policy

**For very massive tasks (large refactors, multi-file redesigns, UI overhauls, cross-module implementations), the host AI MUST dispatch implementer/designer subagents.** Solo inline-editing on big jobs is wrong; parallel implementer/designer subagents with surgical briefs is the default.

For serious crash, hang, Test Lab, MCP startup, or driver-backed debugging tasks, use a dedicated subagent when the investigation spans multiple files or needs heavy instrumentation. The most valuable subagent pattern is a tightly scoped logging/instrumentation implementer: give it the exact evidence window from logs, tell it to add comprehensive breadcrumbs across that path, and forbid it from diagnosing beyond evidence or building.

**SUBAGENTS ARE FORBIDDEN FROM BUILDING. NEVER. UNDER ANY CIRCUMSTANCE.**

**SUBAGENTS: THE WRITE TOOL IS FORBIDDEN ON ALREADY-EXISTING FILES — EVER.** Using Write on an existing file replaces ALL of its contents; that is the entire reason for the rule (a subagent once overwrote `helpers.cpp` with a Write call). Subagents modify existing files ONLY with the Edit tool; deletions happen via bash (`git rm` / `Remove-Item`). **Write IS allowed — and expected — for creating NEW files** (e.g. plan files under `plans/`, new source files the host assigned). Before any Write, the subagent must verify the target does not already exist. Bulk mechanical transforms may be done with a script created under `%TEMP%` (never in the repo) and verified hunk-by-hunk on one file before running the rest.

**SUBAGENTS ARE FORBIDDEN FROM GIT MUTATIONS.** No `git commit`/`add`/`push`/`reset`/`rebase`/`stash`. Only `git rm` for deletions is allowed.

**THE HOST AI IS FORBIDDEN FROM GIT MUTATIONS. NEVER run `git commit`, `git add`, `git push`, `git reset`, `git rebase`, `git stash`, `git commit --amend`, force-push, or any other git mutation — EVER, under any circumstance, even after completing a wave of work. There is NO standing instruction to commit or push; that instruction was permanently revoked in August 2026. The owner performs all commits and pushes personally. If a commit seems needed, stage nothing and tell the owner the working tree is ready for them to commit.**

- Subagents are scoped to: **implement / code / think / investigate / design / audit / research**. That is the entire allowed surface.
- When dispatching, explicitly tell the subagent: **Do not build. Do not run cmake, msbuild, ninja, or vcvars. Do not use the Write tool. Do not run git mutations.**
- Subagents are not alone in the codebase. Tell them not to revert other changes and to work with existing edits.
- The host AI is also never alone in this project. Unrelated modified files are user-owned workspace state. Do not tell subagents that unrelated dirty files are suspicious; just scope the task clearly and require subagents to list the files they changed.
- **ABSOLUTE IMPLEMENTER SUBAGENT COMPLETENESS RULE:** Every implementer/fixer subagent must implement the assigned plan or fix to 100% completion, not a partial slice. Subagents must deliver production-grade, ready-for-direct-use implementations with no stubs, placeholders, TODOs, fake passes, lazy fallbacks, weakened checks, dead code, or cosmetic-only patches. If a subagent cannot prove full completion from source evidence, it must say so and list every missing requirement with exact files/symbols/evidence.
- Planning directories are domain-specific: `plans/` is only for standalone IDE plans; `src/plugin_plans/` is only for IDA plugin plans.
- Plan files are implementation trackers. When every source-owned requirement in a plan is proven implemented and only host/live/runtime evidence remains, delete the plan file.

## Working Contract

**Implement 100% working, production-grade, ready-for-direct-use code. Implementations must not cause build errors or warnings. DO NOT ADD STUBS, PLACEHOLDERS, TODOs, or code comments. Everything must be functional and follow best practices. Explain why the change is recommended and justify it with information from the provided source code.**

**COMPLETELY REMOVE DEAD CODE FROM THE SOURCE CODE ENTIRELY.** Do not leave unused functions, constants, includes, libraries, branches, compatibility shims, retired-path labels, or stale tests after a replacement or removal.

**ALWAYS PERFORM A VERIFICATION LOOP: Plan -> Execute -> Verify -> Report.**

- Ask concise questions before making risky assumptions. If there are no blocking questions, proceed and say so in the report.
- Prefer evidence over speculation. For crashes or "not working" reports, inspect logs first. `aida_debug.log` is the canonical evidence source for app/chat/driver state.
- If something is crashing or behavior is unclear, add comprehensive debug logging in the narrow path, rebuild, and ask the user to run the rebuilt binary and paste the output.
- For crashes, BSODs, hangs, startup stalls, loader stalls, and any other not-working report, do not diagnose from code structure, intuition, or PDB/symbol guesses alone. Treat logs, dumps, and explicit breadcrumbs as the source of truth.
- Crash logging must capture entry/exit, PID, TID, timestamps, elapsed durations, phase/step names, module base/end, VA/RVA/size/protection/state for memory operations, Win32 last-error/NTSTATUS values, IOCTL inputs/results, and before/after state for guard pages, `VirtualProtect`/`NtProtectVirtualMemory`, driver calls, and background work items.
- Do not claim root cause for crashes unless the logs or dump prove it. Use "first confirmed failure marker" or "current evidence window" when the evidence is not yet conclusive.
- Debug logs and diagnostic breadcrumbs are intentionally retained and must not be removed, reduced, or hidden unless the user explicitly asks.

## Session Change Discipline

Every session must be careful with every change. Treat the existing worktree as shared state, assume unrelated modifications are intentional, and keep edits narrowly scoped to the requested objective.

- Inspect the relevant files and current worktree before editing. Do not overwrite, normalize, reformat, revert, or regenerate unrelated files.
- **ABSOLUTE WORKTREE RULE:** If a file is modified and the host or subagents were not explicitly tasked to work on that exact file, ignore it completely. It is always user-owned work.
- Keep changes auditable. A future reviewer should be able to tell which invariant was preserved, what evidence justified the edit, and which verification proved it.

## User Operating Preferences

- The user strongly prefers evidence-driven work over theories.
- Do not add only a few debug logs, build, ask for a run, then repeat the same cycle. For confirmed crash/hang windows, instrument the whole narrow subsystem deeply enough to identify entry, exit, state, timing, and failing call in one reproduction whenever possible.
- When the problem is broad or high-stakes, dispatch a subagent specifically for comprehensive debug logging or implementation, then the host reviews, patches if needed.
- Creating small local test apps, focused repro harnesses, or API-behavior probes is encouraged when it can safely validate a low-level change.
- The user wants AiDA to be extremely stable. Do not break the driver bridge, the debugger, or the message pump for convenience.
- When adding debug logs, prefer complete diagnostic capture over cosmetic sanitization. Keep raw credentials, private keys, OAuth bearer tokens, and API keys out of logs unless the user explicitly directs a controlled local diagnostic capture.

## Confirmed Diagnostics Lessons

- `aida_debug.log` is the canonical app/runtime log; `C:\Users\Public\Desktop\aida_kernel.log` is the kernel-side companion; `C:\Users\Public\Desktop\aida_full_test.log` is the Test Lab full-run evidence source. Read all relevant logs before code diagnosis.
- `AiDAStandalone.exe` crashes generate WER dump files under `C:\CrashDumps\`, for example `C:\CrashDumps\AiDAStandalone.exe.<pid>.dmp`. For standalone crash work, inspect the newest matching dump along with `%TEMP%\aida_bootstrap.log` and Event Log before changing code.
- Test Lab failures can cascade from a single bad target launch. If full-test logs show `target_unavailable=1`, `target_pid=0`, invalid PID/DTB, failed memory allocation, or driver attach mismatch, verify `AiDA_TestTarget.exe` launch and attach first.
- A confirmed full-test launch failure was `CreateProcessW` with `gle=267` because `cwd='AiDA_TestTarget.exe'`. When launch fails, log requested/effective executable path, requested/effective working directory, command line, PID/TID, elapsed time, Win32 error, and driver attach state.
- TCP/network tests can appear stuck when tracker shutdown is not bounded. TCP tracker tests must log before/after start, before/after stop, worker enter/exit, cancellation state, timeout length, elapsed time, and whether `driver_bridge::get_captured_packets` returned.
- MCP may bind a port while still failing clients. If a client times out awaiting `tools/list`, verify live `/health`, `/mcp`, and `tools/list`, inspect `netstat` state, and check for `mcp_srv` `request_entry`, route-specific handler logs, and `request_exit`.
- Standalone MCP exposes mutating tools through localhost. Keep the server responsive without using the shared app work queue for long-lived HTTP/SSE server work; preserve request/stream limits, cancellation, and shutdown diagnostics.
- Before building when `AiDAStandalone.exe` is running, try to close it gracefully. If Windows denies termination, attempt the canonical build anyway and let the build logs prove whether the binary lock matters.
- Standalone UI freezes that show `AiDAStandalone.exe` still running, render/heartbeat logs still advancing, `IsHungAppWindow=True`, and `SendMessageTimeout(WM_NULL)` failing are message-pump responsiveness failures until proven otherwise. Do not blame Camoufox merely because browser work happened near the freeze.
- Preserve the standalone Win32 message pump invariants in `src/standalone/src/main.cpp`. `kAidaQueuedPeekFlags` must include `PM_QS_SENDMESSAGE`, send-only pending work must be drained with `PM_REMOVE | PM_QS_SENDMESSAGE`, and the empty-queue path must still perform a nonblocking `PeekMessage` probe instead of breaking immediately after `GetQueueStatus(QS_ALLINPUT) == 0`. Subagent changes have regressed this before and caused the IDE to load, respond briefly, then become unresponsive while rendering continued.
- When subagents touch standalone startup, rendering, source reconstruction, Camoufox integration, dialogs, or any code near the Win32 loop, the host must review the final diff for the message-pump invariants above. A successful compile is not enough; verify with logs or a live run that the rebuilt `AiDAStandalone.exe` remains responsive after the IDE loads.

Useful files:
1. `C:\Users\ruar1337\AiDAPrivate\strip_comments.py` to strip comments from a directory
2. `C:\Users\ruar1337\AiDAPrivate\scan_libs.py`, `inspect_pe.ps1`, `analyze_logs.ps1`, `find_sym.ps1` — dev utilities

## Mandatory Tool Use

Use Serena for symbol-level code navigation and refactoring:
- Activate the project before code work and read Serena initial instructions when needed.
- Prefer `get_symbols_overview`, `find_symbol`, `find_referencing_symbols`, `find_declaration`, and targeted `search_for_pattern` over broad full-file reads.
- Use Serena symbolic edits for whole-symbol changes where appropriate; use normal patches for small local line edits.
- Keep Serena memories current with durable project facts.

Use Context7 for library/API documentation:
- When external library or API behavior matters, resolve the library ID with Context7 first, then query docs for the relevant version.
- Use Context7 before nontrivial changes involving CMake, ImGui, Zydis/Capstone, OpenSSL, IDA SDK patterns if available, or other library APIs.
- Do not rely on stale memory for current library behavior when docs are available.

## Privacy / Data Rules

- AiDA never phones home. There is no telemetry endpoint, no update server, no license server. User-configured AI provider endpoints (OpenAI/Anthropic/Gemini/OpenRouter/Copilot etc.) and public package registries (npmjs/pypi, for the MCP marketplace) are the only legitimate outbound traffic, plus Camoufox browsing itself.
- Never expose raw credentials, API keys, OAuth tokens, or DPAPI plaintext in chat, commits, docs, or tests. User AI-provider keys are stored DPAPI-obfuscated in `core/auth/auth_store.cpp` and `core/settings/standalone_settings.hpp` — that mechanism stays (it protects the USER's secrets, not AiDA).
- Treat MCP localhost servers as local trust boundaries, not harmless read-only APIs. Some tools mutate IDA state, memory/process state, files, or sessions.

## Driver Rebuild Pipeline

When editing files under `driver/WhosWho/` or `mapper/`:

1. Build the driver `.sln` in Visual Studio/MSBuild x64 Release (outputs to `build-ninja/Release/`), or `.\build-host.cmd -Drivers` when builds are authorized.
2. CMake configure picks up the new binary and the `encrypt_drivers` target regenerates `src/whoswho_embedded.h` (plain embed, no encryption).
3. Full build to link the updated header into AiDAStandalone/AiDA.
4. Tell the user that **reboot is required** to load the updated kernel driver (WhosWho has no unload routine).

## Camoufox Model

- **Camoufox is the only supported browser for AiDA.** Do not add, restore, or prefer Chrome, Edge, Firefox, system-default browser, Playwright-managed stock browser, or any other browser fallback for browser automation, web search, interception, privacy, or reverse-MCP workflows. Camoufox is mandatory for anti-WebRTC and user-agent privacy guarantees.
- Camoufox discovery checks the repo-local bundle first, especially `C:\Users\ruar1337\AiDAPrivate\camoufox-135.0.1-beta.24-win.x86_64\camoufox.exe`, then existing build/dependency fallbacks.
- Do not add fileless loader code, in-memory PowerShell PE mapping, `AIDA_FILELESS_LAUNCH`, or `Invoke-AidaPEInMemory`.

## Key Patterns and Conventions

- **Driver API**: user-mode `voyager::device_t` in `driver/comm.h`/`driver/comm.cpp`; static IOCTLs (`0x00220000 | ((0x800+offset)<<2)`), fixed device name `\\.\WhosWho`; shared global `inline std::unique_ptr<voyager::device_t> device`. User/kernel ABI structs in `comm.h` and WhosWho `Struct.h` must stay synchronized (packing, field order, sizes, static asserts).
- **MCP tool pattern**: Tools register with a JSON schema: name, description, params, `read_only`, handler. Cross-instance routing uses `instance_id`/`pid`.
- **MCP tool consolidation discipline**: Do not inflate MCP tool count with duplicates or CRUD/action redundancies. Prefer consolidated manage-style tools with an explicit `action` parameter. Do not expose manual tools for behavior that should run automatically in the backend/UI/LLM flow.
- **ImGui rendering**: DirectX11 backend with FreeType font rendering, DWM blur, DPI-aware scaling.
- **Event bus**: Type-safe publish/subscribe in `core/infra/event_bus.cpp` for inter-module decoupling.
- **Session persistence**: SQLite for chat messages, analysis sessions, and cost tracking; JSON for config and graph data.
- **No comments**: Do not add code comments, docstrings, or TODOs. CMake `COMMENT "..."` strings are acceptable because they are command metadata.

## Directory Guide

### `driver/`

Purpose: WhosWho KMDF driver (functional RE features only) + shared user-mode communication/test code. The Sentinel protection driver was deleted; do not recreate it.

Key files:
- `driver/comm.h` / `driver/comm.cpp`: user-mode `voyager::device_t` API and driver connection logic (static IOCTLs, no handshake/session crypto).
- `driver/test_driver.cpp`: integration-style exerciser for connection, memory, process, and network APIs.
- `driver/WhosWho/WhosWho/src/DriverEntry.cpp`: driver entry, fixed device/symlink, dispatch init, network capture + MalwareSafe (target sandboxing) startup.
- `driver/WhosWho/WhosWho/src/function/Dispatcher.h`: IOCTL dispatcher. Keep user/kernel ABI structs synchronized with `driver/comm.h`.
- `driver/WhosWho/WhosWho/src/function/impl/`: Memory.cpp, DTB.cpp, BaseAddress.cpp, Network.cpp (WFP capture/DNS/inject/redirect/pcap), RemoteCall.cpp, Debugger.cpp (thread ctx, HW breakpoints), physical-memory helpers.
- `driver/WhosWho/WhosWho/src/function/MalwareSafe.h`: sandboxing of ANALYSIS TARGETS (a feature, not self-protection).
- `driver/WhosWho/WhosWho/src/function/KernelDebugCapture.h/.cpp`: writes `C:\Users\Public\Desktop\aida_kernel.log`.
- `driver/analysis/`, `driver/exploit/`, `driver/RULES.md`, `driver/LOOK_FOR.md`: the owner's personal kernel-research material (untracked); never touch unless asked.

Rules:
- Kernel code uses `ExAllocatePool2`, pool tags, `RtlSecureZeroMemory`, SEH, explicit IRQL checks, and requestor-mode checks. Preserve those patterns.
- Never touch paged-pool memory while holding a spin lock. Use nonpaged scratch buffers for data touched under the lock.
- Do not run, install, deploy, load, or unload drivers on the host unless explicitly requested.
- Do not reintroduce self-protection, anti-debug, anti-dump, hiding, attestation, heartbeat, or server contact into the driver.

### `mapper/`

Purpose: WindMapper manual-map loader that loads WhosWho.sys. CLI: `WindMapper.exe <whoswho.sys> [shadowfs.sys]`. `src/driver_loader.cpp` invokes it first, with a plain NtLoadDriver/registry-service fallback.

### `src/`

Purpose: Windows C++ implementation for the IDA Pro plugin plus shared code. The plugin exposes IDA analysis, decompilation, vulnerability analysis, GraphRAG, and mutation tools over MCP.

Key modules:
- `src/aida.cpp` / `src/aida.hpp`: IDA plugin lifecycle, actions, MCP startup. No EULA, no gating, no kill paths.
- `src/agent_tools.cpp` / `src/agent_tools.hpp`: MCP tool registry and implementations, including mutating tools such as patch/rename/type/comment and `execute_python`.
- `src/mcp_server.cpp` / `src/mcp_server.hpp`: localhost MCP server, SSE/HTTP endpoints, multi-IDA registry, client config writing.
- `src/settings.cpp` / `src/settings.hpp`: local config and provider keys (DPAPI/`enc1:` obfuscation for USER API keys stays).
- `src/driver_loader.cpp`: stages the plain embedded WhosWho.sys and starts the mapper process (NtLoadDriver fallback).
- `src/aida_ipc.cpp`: `trace_breadcrumb` diagnostic logger only.
- `src/ida_utils.*`, `src/analysis_db.hpp`, `src/graphrag.*`, `src/emulation_engine.*`: IDA context extraction, persisted analysis/chat data, graph context, driver-backed emulation.
- `src/vuln/*`: vulnerability analysis engines and tools; symbolic/SMT/Z3 verification paths.

Rules:
- Preserve IDA SDK/Hex-Rays API patterns. Strings are plain (`OBFSTR` was removed; do not reintroduce obfuscation).
- Do not run the plugin, driver loader, or hypervisor-detector paths casually.
- Treat API keys and session data as secrets.
- MCP binds to localhost but exposes mutating tools and permissive CORS. Treat it as a local trust boundary.

### `src/standalone/src/`

Purpose: Windows standalone AiDA client: Win32/DX11/ImGui desktop app with AI chat, MCP tools, debugger/disassembler/scanner/network tooling, and test-lab surfaces. It boots straight into the IDE; there is no license screen and no startup gate of any kind.

Key files:
- `src/standalone/src/main.cpp`: app entry, Win32/DX11/ImGui setup, font loading, DPI/acrylic windowing, embedded Z3 load, background initialization, render loop.
- `src/standalone/src/helpers/helpers.cpp` / `.h`: app chrome, tabs, input gating, central UI rendering.
- `src/standalone/src/helpers/globals.h`: shared UI/app state, center view routing, chat state, editor/file helpers, persistence bridges.
- `src/standalone/src/core/tools/standalone_tools_fwd.hpp`: registration surface for MCP/tool domains.
- `src/standalone/src/core/mcp/mcp_standalone.cpp`: local MCP server, bound to `127.0.0.1`.
- `src/standalone/src/core/settings/standalone_settings.hpp`: settings schema, provider profiles, sandbox/MCP config, DPAPI/fallback obfuscation for user secrets.
- `src/standalone/src/core/ui/embedded_resources.hpp` / `.cpp`: Ghidra spec extraction. The four specs ship in qrc (`src/standalone/resources/aida_ghidra.qrc.in` → `:/ghidra/{x86-64.sla,x86-64.pspec,x86-64-win.cspec,x86.ldefs}`, configured/compiled via `qt6_add_resources` in CMakeLists.txt); `embedded_resources.cpp` reads them via `QResource::uncompressedData()` and writes the same temp-dir files (sleigh needs filesystem paths). The c03 closure keeps its own Qt-free `embedded_resources::extract_ghidra_specs()`/`ghidra_spec_resource_count()` definitions in `tests/c03/testlab_runtime/safe_headless_runtime_bridges.cpp` (faithful "no embedded specs in this module" behavior). `libz3.dll` staging is the separate POST_BUILD copy step (not part of these files).

Rules:
- Windows-first C++ with many Win32 APIs, `#pragma comment(lib, ...)` dependencies, and header-heavy modules.
- Use existing `work_queue` for background work; startup offloads chat, network, scanner, MITM, script engine, and driver bridge work.
- Preserve SEH wrappers and diagnostic logging patterns around render/init calls.
- UI state is largely global. Prefer extending existing `globals::ui`, `helpers`, and `core/ui` patterns instead of adding parallel state systems.
- `core/testlab` includes driver tests with serious side effects. Inspect before running.

### `src/standalone/src/core/`

Purpose: standalone core for ImGui UI, AI/provider/auth flows, MCP server/tools, reverse engineering, disassembly, debugger, scanner, network/Burp-style tooling, session persistence, and in-app tests.

Key entry points:
- `core/mcp/mcp_standalone.hpp`: MCP tool results, visibility, `server_t`, target scoping.
- `core/mcp/mcp_standalone_tools.cpp`: standalone MCP tool registration and fan-out to domain registrars.
- `core/network/burp/burp_module.cpp`: Burp-style module and MCP registration.
- `core/session/session_store.cpp`: SQLite schema and session DB location logic.
- `core/settings/standalone_settings.hpp`: config path and secret storage/obfuscation.
- `core/auth/auth_store.cpp`: DPAPI storage for the USER's AI-provider auth/API material (user-secret storage — keep).
- `core/runtime/standalone_driver.hpp` / `.cpp`: the `driver_bridge` — user-mode front end to WhosWho (memory R/W, modules/threads, contexts/HWBP, remote calls, network capture, sandboxing, debug events). Driver load failure is non-fatal; RE tools gate on `is_loaded()`.
- `core/runtime/kernel_symbols.hpp` / `.cpp`: in-memory kernel PDB symbol engine backed by the vendored `.deps/MemPDB` static library (C++20; only this TU compiles as C++20). Resolves/downloads `ntoskrnl` symbols from the live kernel image's RSDS record, caches PDBs under `%LOCALAPPDATA%\AiDA\Standalone\symbols`, and backs the kernel-memory tools plus the `driver_kernel_symbols` manage tool.
- `core/analysis/flirt/`: offline static library-code recognition: FLIRT signature DB, anchored prefix-indexed parallel matcher, post-baseline `static_recognition_service`, and `type_seed_exporter`. Static RTTI/vfunc-slot scans live in `core/re/rtti.*` and `core/re/vmt.*`.
- `core/testlab/test_lab.hpp`, `core/testlab/test_lab_view.cpp`: in-app test registration and work-queue execution (functional suites only).

Rules:
- Long-running work should use `core/infra/work_queue.hpp`; initialize/shutdown pairs must stay balanced.
- Existing patterns prefer namespaces plus static module state, `std::atomic`/mutex guards, and `diag::log_tagged*` logging.
- MCP tools mark mutating operations `read_only=false`; internal filesystem/web tools use internal visibility.
- Preserve cancellation tokens and shutdown draining for MCP, proxy, scanners, and test workers.
- Do not log raw API keys, OAuth access/refresh tokens, DPAPI plaintext, TLS keys, or captured traffic bodies.
- Be careful around driver attach/read-memory paths, TLS key extraction, MITM proxy/cert generation, sandbox execution, local command execution, and internal file tools.

## Completion Checklist

Before final response:
- Confirm no unrelated user changes were reverted.
- Confirm secrets were not printed or modified.
- Confirm generated files were not hand-edited (`src/whoswho_embedded.h` is generated by `src/encrypt_whoswho.py`).
- When the user authorizes a build, run the canonical build and confirm zero errors and zero new warnings.
- For driver changes, follow the driver rebuild pipeline and always remind the user that reboot is required to load the updated kernel driver.
- For library/API changes, mention Context7 docs consulted when relevant.
- For symbol-level code work, mention Serena navigation/refactoring used when relevant.
- Report any checks that could not be run.
- NEVER commit or push. All git mutations are forbidden for the host AI and subagents; the owner commits and pushes personally.

---
> Source: [sigwl/AiDAPrivate](https://github.com/sigwl/AiDAPrivate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->

## pc98

> This is a Windows-focused PC-98 launcher and translation wrapper. The PC-98

# Agent Notes

## Project Shape

This is a Windows-focused PC-98 launcher and translation wrapper. The PC-98
catalog entrypoint is `game-launcher.bat`; the universal standalone-window
translator entrypoint is `universal-launcher.bat`. Implementation code lives in
`scripts/`.

Important local folders:

- `emulator/`: user-provided Neko Project 21/W files.
- `games-rard/`: user-owned `.rar`/`.zip` game catalog.
- `disks/`: extraction cache and user disk images.
- `models/`: local GGUF and mmproj model cache.

Do not commit emulator binaries, BIOS files, commercial game archives, extracted
disk images, screenshots, downloaded models, or local launcher state.

## Main Workflows

- Launcher: `python scripts\game-launcher.py`
- Batch entrypoint: `.\game-launcher.bat`
- Universal window launcher: `.\universal-launcher.bat`
- Direct emulator runner: `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run-pc98.ps1`
- Translator: `python scripts\translate-screenshot.py --watch`
- Local LLM bootstrap: `python scripts\local_llm.py status`

The launcher scans `.rar` and `.zip` files under `games-rard/`, extracts
selected archives into `disks\catalog\`, launches Neko Project through
`run-pc98.ps1`, starts a local `llama-server` when needed, and opens the
translation watcher.

The universal launcher lists visible windows and starts the watcher with
`--hwnd`, so standalone Windows games can be translated without going through
the PC-98 catalog launcher.

## Launcher State

`launcher-state.json` is local runtime state and ignored by git. It stores:

- `recent`: recently launched games with play counts.
- `favorites`: archive paths toggled with `Space`.

Favorites are displayed with ASCII `*`. Keep state loading backward compatible:
missing or invalid keys should degrade to empty lists.

## Translation Runtime

LM Studio is no longer required. `scripts/local_llm.py` manages a local
OpenAI-compatible `llama-server` on `http://127.0.0.1:8080/v1` by default.

The image translation path requires both:

- `Gemma-4-E4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf`
- `mmproj-Gemma-4-E4B-Uncensored-HauhauCS-Aggressive-f16.gguf`

Both live under ignored `models/` by default. The launcher may stop only the
`llama-server` process it started itself; it should not kill an already-running
server owned by the user.

The launcher should start `llama-server` with GPU offload when a device is
available. It prefers ignored local CUDA runtime
`runtimes\llama-cuda\llama-server.exe` over winget when present. Device
selection is automatic by default, preferring CUDA, then NVIDIA/Vulkan, then the
first reported device. Respect `LOCAL_LLM_DEVICE`, `LOCAL_LLM_GPU_LAYERS`,
`LOCAL_LLM_SERVER_PATH`, and `LOCAL_LLM_SERVER_ARGS` overrides. Do not commit
runtime DLLs/exes under `runtimes/`.

## UI Conventions

Keep the launcher console-native and dependency-free. The current style uses:

- ANSI colors with a plain-text fallback.
- ASCII-only panels and markers.
- Best-effort Windows console font setup.
- Arrow keys for navigation, Left/Right for pagination, `Space` for favorites,
  `Enter` for launch/select, and Esc/Backspace for back.

## Verification

Before finishing launcher or translator changes, run:

```powershell
python -m py_compile .\scripts\game-launcher.py .\scripts\translate-screenshot.py .\scripts\local_llm.py
```

For launcher-only state changes, also run a small import/state smoke test or an
equivalent non-interactive check that old state files still load.

Use `git status --short` before committing. Keep generated caches and local
runtime artifacts out of commits.

## Security Notes

- Validate archive member paths before extraction; reject absolute paths,
  traversal, control characters, and links.
- Keep managed `llama-server` binds loopback-only unless
  `LOCAL_LLM_ALLOW_REMOTE_BIND=1` is explicitly set.
- Keep GGUF/mmproj downloads pinned to expected SHA256 hashes.
- Do not let `LOCAL_LLM_SERVER_ARGS` override model, host, port, alias, device,
  or GPU-layer settings.
- Watcher cleanup should only stop launcher-managed `llama-server` processes,
  not arbitrary PIDs.

---
> Source: [b2kdaman/pc98](https://github.com/b2kdaman/pc98) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->

## tts-studio-pro

> Project-specific notes:

# Agent Instructions

Project-specific notes:

- Primary product: TTS Studio Pro, a TanStack Start/Vite React app for OmniVoice TTS, batch generation, voice library, history, and local Whisper transcription.
- Frontend source lives under `src/`; user-facing routes are in `src/routes/`; shared state is in `src/lib/tts-store.tsx`.
- Local backend lives under `OmniVoice-master/`; `api_server.py` exposes OmniVoice TTS and local OpenAI Whisper transcription APIs on port `8001`.
- Use `npm.cmd` on Windows instead of `npm` because PowerShell script execution policy may block `npm.ps1`.
- For backend Python checks, use `OmniVoice-master/.venv/Scripts/python.exe` when available.
- Do not edit generated `dist/` assets or `src/routeTree.gen.ts` by hand; let build/router tooling regenerate them.
- `harness-experimental/` is the cloned upstream Harness source. The installed Harness operating files for this app are `AGENTS.md`, `docs/`, `scripts/`, and `harness.db`.

Windows Harness CLI note:

- The upstream installer does not publish a Windows Harness CLI release asset yet. This repo has a locally built CLI at `scripts/bin/harness-cli.exe`.
- In PowerShell, prefer `scripts\bin\harness-cli.exe <command>`.
- In Git Bash, `scripts/harness <command>` should work because `scripts/bin/harness-cli` is also present.

<!-- HARNESS:BEGIN -->
## Harness

This repo uses Harness. Before work, read:

- `README.md`
- `docs/HARNESS.md`
- `docs/FEATURE_INTAKE.md`
- `docs/ARCHITECTURE.md`
- `scripts/harness query matrix` or, on PowerShell, `scripts\bin\harness-cli.exe query matrix`

Use the Rust Harness CLI as the main operational tool. Run it through the
stable repo-local entrypoint `scripts/harness`, which uses the prebuilt Rust
binary at `scripts/bin/harness-cli` in installed projects.
<!-- HARNESS:END -->

---
> Source: [magicwand1993/tts-studio-pro](https://github.com/magicwand1993/tts-studio-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->

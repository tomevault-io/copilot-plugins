## codex-account-switch

> - Windows and macOS should be developed independently whenever practical.

# Repository AGENTS

## Platform Isolation

- Windows and macOS should be developed independently whenever practical.
- Do not introduce shared modules or shared runtime paths unless the behavior is a necessary cross-platform contract, disk format, or build/versioning concern.
- Do not extract or expand shared modules for platform-specific behavior unless the shared code is strictly necessary for identical cross-platform contracts, on-disk formats, or version/build plumbing.
- When syncing a feature from one platform to the other, prefer implementing or adapting it under `src-tauri/win/**` or `src-tauri/mac/**` first. Use `src-tauri/shared/**` only for unavoidable neutral contracts.
- For Windows-only changes, use Windows-only compile, check, test, and build commands; do not run validation paths that require compiling `src-tauri/mac/**` unless the task explicitly touches macOS.
- Windows Vite/Tauri build commands should be run with escalation by default. In this repository, sandboxed builds are known to fail with `spawn EPERM` while starting `esbuild` from the Vite/Tauri `beforeBuildCommand`.

---
> Source: [Cmochance/Codex_Account_Switch](https://github.com/Cmochance/Codex_Account_Switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->

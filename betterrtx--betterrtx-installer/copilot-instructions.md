## betterrtx-installer

> - Use **Tauri v2** APIs and docs; prefer NSIS installer on Windows.


# Project Rules — BetterRTX Installer (Tauri v2)

- Use **Tauri v2** APIs and docs; prefer NSIS installer on Windows.
- Frontend: React + TS; strict mode; Tailwind CSS v4; keep UI edits isolated. Reference Tailwind @docs
- Rust core: small commands with `#[tauri::command]`; pure helpers.
- use context7 to referecne Tauri docs
- Test with `bun run tauri dev` from the `.v3/` directory. Do not use NPM.
- Prefer Tauri plugins over implementing own features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BetterRTX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->

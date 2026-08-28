## claude-switch

> Windows tray app (Tauri v2) for multi-account Claude Code usage + switching. See `README.md` (what/how), `ARCHITECTURE.md` (structure + invariants), `docs/RESEARCH.md` (endpoint research with sources).

# Claude Switch working rules

Windows tray app (Tauri v2) for multi-account Claude Code usage + switching. See `README.md` (what/how), `ARCHITECTURE.md` (structure + invariants), `docs/RESEARCH.md` (endpoint research with sources).

## Commands

- `npm run tauri dev` — dev run
- `npm run tauri build` — release build (exe in `src-tauri/target/release`)
- `cargo check` in `src-tauri/` — fast compile check

## Hard rules

- NEVER JSON-parse-and-rewrite `~/.claude.json` (duplicate keys). String surgery only, via `switcher.rs`.
- NEVER log, print, or commit tokens. The vault is DPAPI-encrypted; keep it that way.
- Any change to refresh/polling behavior must respect the invariants in ARCHITECTURE.md (rotated refresh tokens persisted immediately; 120s minimum poll; 429 backoff).
- Design tokens live in `ui/styles.css` (`:root` variables); don't invent new colors.

## Gotchas

- `tauri.conf.json` needs `withGlobalTauri` (frontend has no bundler).
- Tray icon updates use `Image::new_owned` raw RGBA, drawn by `trayicon.rs` (no font deps, hand-coded 5x7 digits).
- Real user credentials are involved when testing switch/capture; backups land in `%APPDATA%\claude-switch\backups\`.

---
> Source: [Sahir619/claude-switch](https://github.com/Sahir619/claude-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->

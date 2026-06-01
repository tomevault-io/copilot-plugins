## promptbridge

> PromptBridge is a Windows/macOS desktop app (Tauri + React + Rust) that translates non-English coding requests into English prompts for AI coding agents. It is BYOK and runs fully locally.

# AGENTS.md — PromptBridge

## Identity

PromptBridge is a Windows/macOS desktop app (Tauri + React + Rust) that translates non-English coding requests into English prompts for AI coding agents. It is BYOK and runs fully locally.

## Behavior Rules

- Do not add real user or customer data.
- Do not add auth, cloud sync, subscription, payment, or server-side translation unless explicitly approved.
- Do not add new provider integrations unless explicitly approved.
- Do not modify the protection rule logic (code/command/path/URL token guards) without an explicit spec.
- Do not change the overlay window lifecycle (show/hide/shortcut) behavior without an explicit plan.
- Mark unknowns as **To Confirm**.
- Mark unapproved decisions as **Pending**.
- Implement one approved slice at a time.
- Preserve all existing working flows before adding new behavior.
- Run verification commands after any code change.
- Do not commit unless explicitly asked.

## Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, TypeScript, Tailwind CSS v4, Vite 8 |
| Desktop shell | Tauri v2 |
| Backend | Rust (src-tauri/src/lib.rs) |
| State persistence | @tauri-apps/plugin-store |
| Clipboard | @tauri-apps/plugin-clipboard-manager |
| Shortcuts | @tauri-apps/plugin-global-shortcut |
| Autostart | @tauri-apps/plugin-autostart |
| Tests | Vitest |

## Source of Truth

- **AGENTS.md** — behavior rules (this file)
- **docs/PROJECT_STATE.md** — current state, active feature, implementation lock
- **docs/VERIFICATION.md** — verification commands and pass/fail criteria
- **docs/DEV_LOG.md** — completed history
- **docs/SOURCE_OF_TRUTH_INDEX.md** — document authority index

## Guardrails

| Area | Rule |
|---|---|
| Provider credentials | Never log, commit, or expose API keys |
| Protection rules | Token guards must survive any translation pipeline change |
| Paste injection | Must restore original clipboard after inject |
| Overlay | Must hide on Escape and on successful inject |
| Local history | Must respect the Save local history toggle |
| Privacy guards | Sensitive-data warnings must fire before any provider call |

---
> Source: [MinkyuTheBuilder/PromptBridge](https://github.com/MinkyuTheBuilder/PromptBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->

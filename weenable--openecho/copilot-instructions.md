## openecho

> OpenEcho is a macOS desktop app for voice-to-text with five modes: Input, Translate, Question, Memo, and Voice Generate. Built with Tauri v2 (Rust backend + React/TypeScript frontend).

# OpenEcho - Claude Code Project Guide

## Project Overview

OpenEcho is a macOS desktop app for voice-to-text with five modes: Input, Translate, Question, Memo, and Voice Generate. Built with Tauri v2 (Rust backend + React/TypeScript frontend).

## Architecture

- **Frontend**: React + TypeScript + Tailwind CSS v4 (`src/`)
- **Backend**: Rust + Tauri v2 (`src-tauri/src/`)
- **Speech recognition**: whisper-rs (local, runs on device)
- **TTS**: Kokoro (local) or OpenAI-compatible API (remote)
- **LLM**: OpenAI-compatible API for polish, translate, question, memo modes

## Key Files

- `src/Overlay.tsx` — Floating overlay UI (recording, results, TTS)
- `src/MainWindow.tsx` — Main settings/history window
- `src/SetupFlow.tsx` — First-launch setup wizard
- `src/LangPicker.tsx` — Language selection popup
- `src/lib/i18n.ts` — All UI translations (10 languages)
- `src/lib/tauriCommands.ts` — Frontend wrappers for Rust commands
- `src-tauri/src/lib.rs` — Main Rust entry, Tauri commands, app lifecycle
- `src-tauri/src/transcribe.rs` — Whisper transcription + hallucination filtering
- `src-tauri/src/llm.rs` — LLM API calls (polish, translate, question, memo)
- `src-tauri/src/tts.rs` — TTS generation (local Kokoro + remote API)
- `src-tauri/src/audio.rs` — Audio capture via cpal
- `src-tauri/src/config.rs` — App config persistence (~/.openecho/)

## Build Commands

```bash
npm install              # install frontend deps
npm run dev              # vite dev server (frontend only)
npm run build            # build frontend (tsc + vite)
npm run tauri dev        # full dev mode (frontend + Rust)
npm run tauri build      # production build (.dmg)
cargo check --manifest-path src-tauri/Cargo.toml  # check Rust only
```

## Conventions

- UI text must go through `src/lib/i18n.ts` — never hardcode user-facing strings
- Tauri commands defined in Rust (`#[tauri::command]`) must be registered in the `.invoke_handler()` call in `lib.rs`
- Frontend command wrappers go in `src/lib/tauriCommands.ts`
- The overlay window is a separate Tauri window (`#/overlay` route)
- Audio data flows: cpal capture → f32 buffer → 16kHz resample → whisper-rs → LLM polish
- Recordings saved to `~/.openecho/recordings/`, TTS audio to `~/.openecho/tts-audio/`
- Config stored at `~/.openecho/config.json`

## Common Patterns

- Use `invoke("command_name", { args })` to call Rust from frontend
- Use `emit("event-name", payload)` / `listen("event-name", handler)` for Rust↔frontend events
- Overlay modes: recording → transcribing → result (or TTS input → generating → result)
- `busyRef` guards prevent double-submission in overlay actions
- History stored in localStorage (`openecho.history.v2`)

---
> Source: [weenable/openecho](https://github.com/weenable/openecho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

## obs-live-translate

> A native **Windows** OBS Studio plugin that does real-time speech-to-speech

# AGENTS.md

## What this repo is

A native **Windows** OBS Studio plugin that does real-time speech-to-speech
translation via the Google Gemini Live API (`gemini-3.5-live-translate-preview`).
A microphone's audio is streamed to Gemini and the translated audio is emitted
as a separate OBS audio source that can live on its own track.

**The plugin is implemented and released.** See `README.md` for the architecture
and behavior. Your job now is **maintenance and incremental changes**, not a
green-field build.

## Build / test / deploy (Windows x64)

This is a Windows-only build (needs libobs). Do **not** try to build on a
non-Windows filesystem — there is no libobs there. `cmake` from Visual Studio is
not on the non-interactive PATH; prepend it:

```powershell
set "PATH=C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin;%PATH%"
cmake --preset windows-x64                          # configure (first run fetches obs-deps)
cmake --build --preset windows-x64                  # plugin DLL + tests
cmake --build --preset windows-x64 --target unit-tests   # pure-logic tests, no libobs
ctest --test-dir build_x64 --output-on-failure
```

Install: copy `build_x64\RelWithDebInfo\obs-live-translate.dll` to
`C:\Program Files\obs-studio\obs-plugins\64bit\` (OBS must be closed).

## Rules

- **Test-driven.** For any change: write the failing test → confirm it fails →
  write the minimal implementation → confirm it passes → commit. Pure logic lives
  behind the libobs-free `unit-tests` target so it can be tested without OBS;
  changes that touch libobs (`filter.cpp`, `source.cpp`, `translation-session`)
  are verified by a full plugin build.
- **Do not skip verification.** If a build/test can't run because a prerequisite
  (e.g. libobs) is missing, stop and report what is missing — don't claim done.
- **Commits**: author as `weisunglee`, never add Claude/AI trailers. Use a feature
  branch; do not commit directly to `main` unless explicitly told.
- Keep changes scoped. The spec's non-goals (captions, multiple simultaneous
  sessions, encrypted key storage, explicit source-language selection) remain out
  of scope for v1.

## Architecture (quick map)

- `filter.cpp` — mic audio filter: resample to 16 kHz mono → voice-gate → chunk →
  hand to `TranslationSession`.
- `translation-session.*` — shared singleton: WebSocket to Gemini (reconnect +
  backoff), input/output ring buffers, output-ready notification + interrupt.
- `source.cpp` — translated-audio source: event-driven push loop that drains the
  output buffer and pushes to `obs_source_output_audio`, paced by
  `OutputTimestamper` to keep a bounded (~100 ms) scheduling lead.
- `audio-pacing.*`, `audio-convert.*`, `ring-buffer.*`, `backoff.*`,
  `live-protocol.*`, `base64.*` — pure modules, each with a Catch2 test.

## Key technical facts (verified against Google's docs)

- Model: `models/gemini-3.5-live-translate-preview`, WebSocket (TLS) endpoint.
- Input audio to Gemini: 16 kHz, 16-bit PCM, mono, little-endian, 100 ms chunks
  (3200 bytes/chunk).
- Output audio from Gemini: 24 kHz, 16-bit PCM, mono.
- `translationConfig` has only `targetLanguageCode` (BCP-47) and
  `echoTargetLanguage`. **No source-language param** — the model auto-detects the
  input language.
- The output stream is continuous (the model emits even during silence) and does
  **not** send `turnComplete`/`generationComplete`/`interrupted` in this mode.
  Perceived cut-offs at sentence ends are the model's own phrase-boundary cadence,
  not a plugin bug — the plugin's delivery is gap-free.

---
> Source: [weisunglee/obs-live-translate](https://github.com/weisunglee/obs-live-translate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

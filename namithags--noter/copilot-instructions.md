## noter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Noter is an Android voice-notes app: it records audio, transcribes it on-device, and
summarises the transcript on-device. Nothing is sent to a server — the privacy claim in
the README is a design constraint, so prefer offline/on-device solutions over cloud APIs
when adding functionality.

## Technology Stack

- Kotlin, Jetpack Compose (Material 3), Coroutines/Flow
- Room (persistence), WorkManager (background transcription), Navigation Compose
- Vosk `0.3.75` (`com.alphacephei:vosk-android`) — offline speech-to-text
- ML Kit GenAI Summarization `1.0.0-beta1` — Gemini Nano via AICore
- AGP 8.7.3, Kotlin 2.0.20, KSP 2.0.20-1.0.25, Gradle 8.9, JDK 17, minSdk 34 / compileSdk 35

## Build, Test, Run

```bash
scripts/fetch-vosk-model.sh      # REQUIRED once per clone; downloads ~41 MB model
./gradlew assembleDebug
./gradlew test                   # JVM unit tests (Robolectric)
./gradlew connectedAndroidTest   # instrumented tests, needs a device/emulator
```

Requires JDK 17 (AGP rejects newer JDKs) and `ANDROID_HOME` pointing at an SDK with
platform 35.

## Architecture

MVVM over a repository. Recording and transcription are decoupled through WorkManager:

1. `RecordingViewModel.stopRecording()` inserts a `Note` with a placeholder title and
   enqueues `TranscriptionWorker` (pass keys via `TranscriptionWorker.KEY_*`, never
   string literals).
2. `TranscriptionWorker` → `VoskTranscriber` → `PcmAudioDecoder` → writes the transcript
   file → `NoteSummarizer` → updates the Room row.

Key constraints when touching this path:

- **Vosk only accepts 16 kHz mono 16-bit PCM.** `RecordingManager` records AAC at exactly
  that rate/channel count so `PcmAudioDecoder` can decode 1:1, but the decoder still
  downmixes and resamples so older recordings keep working. If you change the recording
  format, check both sides.
- **The Vosk model lives in `assets/`, not git.** `StorageService.sync` requires a `uuid`
  file in the model directory and re-extracts when it changes; `fetch-vosk-model.sh`
  writes it.
- **Summarisation is best-effort.** Gemini Nano only exists on AICore devices (Pixel 8+,
  Galaxy S24+). `NoteSummarizer` returns a `Result` and never throws; a note must remain
  valid with `summary == null`.
- **Release builds are minified.** JNA resolves Vosk bindings reflectively, so new
  reflective dependencies need rules in `app/proguard-rules.pro`.

Dependencies are wired by hand in `MainActivity.NoterApp()` and re-resolved inside
`TranscriptionWorker` (WorkManager constructs workers itself). A DI framework would
remove that duplication.

## Known Issues

- `FileHelper` writes to `Environment.getExternalStoragePublicDirectory(DIRECTORY_DOCUMENTS)`,
  which scoped storage blocks on API 29+. With minSdk 34 these writes fail at runtime;
  transcript/audio storage needs migrating to `MediaStore` or app-scoped directories.
- The README advertises GitHub Actions builds, but no `.github/workflows/` exists.

---
> Source: [NamithaGS/noter](https://github.com/NamithaGS/noter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->

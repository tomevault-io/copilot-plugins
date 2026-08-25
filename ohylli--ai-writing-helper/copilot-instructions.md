## ai-writing-helper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Writing Helper is a Windows system tray utility (C# / .NET 10 / WinForms) that provides AI-powered typo/grammar fixing and voice dictation via global hotkeys. The primary user is blind and uses the NVDA screen reader — accessibility is a core requirement, not an afterthought.

## Build Commands

```bash
dotnet build          # Build the solution
dotnet run --project src/AIWritingHelper   # Run the app
dotnet test --filter "Category!=Integration"   # Run unit tests (default for development)
dotnet test --filter "Category=Integration"    # Run integration tests (only when explicitly requested)
dotnet publish src/AIWritingHelper -c Release --self-contained -p:PublishSingleFile=true  # Self-contained single-file exe (~50 MB, compressed; no .NET install required)
dotnet publish src/AIWritingHelper -c Release --no-self-contained -r win-x64 -p:PublishSingleFile=true -p:EnableCompressionInSingleFile=false  # Framework-dependent single-file exe (~2 MB; requires .NET 10 Desktop Runtime)
```

**Important:** Only run `dotnet test --filter "Category!=Integration"` during normal development. Integration tests hit real APIs and should only be run when the user explicitly asks for them.

Integration tests require a `.env` file in `tests/AIWritingHelper.Tests/` (copy `.env.example` and add your API key). Tests are skipped automatically if the key is missing.

## Architecture

```
src/AIWritingHelper/
  Core/       # Business logic and provider interfaces
  Services/   # LLM (OpenAI-compatible API) and STT (ElevenLabs Scribe v2) implementations
  UI/         # WinForms settings dialog, system tray icon
  Audio/      # NAudio microphone recording, system sound playback
  Config/     # YAML settings persistence (%APPDATA%\AIWritingHelper\)
tests/AIWritingHelper.Tests/   # xUnit or NUnit tests
```

**Key patterns:**
- Provider abstraction via interfaces for both LLM and speech-to-text services
- Dependency injection for testability — core logic is separated from UI/hardware
- Single concurrent operation enforced (typo fix or dictation, not both)
- Single-instance app enforcement

**Data flows:**
- Typo fixing: clipboard text → LLM API → corrected text back to clipboard → success sound
- Dictation: hotkey toggle → NAudio record → STT API → optional LLM cleanup → text to clipboard or direct paste via SendInput

## Accessibility Requirements

- All WinForms controls need proper accessible names/descriptions
- Logical, complete tab order — every control keyboard-reachable
- No visual-only information; all feedback via audio cues and balloon tip notifications
- Use standard WinForms controls (built-in accessibility support)
- Use `SendInput` API (not `SendKeys`) for simulated input — more reliable with NVDA

## External Services

- **LLM:** OpenAI-compatible API format (configurable base URL, API key, model). Default provider: Cerebras
- **STT:** ElevenLabs Scribe v2 with auto language detection
- Both use API timeouts

## Settings

Stored as YAML in `%APPDATA%\AIWritingHelper\`. Includes API credentials, hotkey bindings, system prompt (user-editable with sensible default), microphone selection, output mode, dictation cleanup toggle and prompt, log level.

## Current Status

Typo fixing and dictation are fully functional end-to-end, including
direct-insertion output mode.

**What's working:**
- System tray app with Settings dialog (General, Typo Fixing, Dictation tabs) and global hotkeys
- Typo fix flow: Ctrl+Alt+Space → clipboard text → LLM API → corrected text back to clipboard → success sound
- Dictation flow (clipboard mode): Ctrl+Alt+D → record → Ctrl+Alt+D → STT → transcript on clipboard → success sound. Empty transcript surfaces "No speech detected" without clobbering the clipboard.
- Dictation flow (direct-insertion mode): same toggle, but the transcript is pasted into the focused window via synthetic Ctrl+V, then the previous clipboard text is restored after a 150 ms settle delay.
- Optional LLM cleanup of dictation: when `DictationCleanupEnabled` is set, the transcript goes through `ILLMProvider` with `DictationCleanupPrompt` (shares the typo-fixing endpoint/key/model) before output. Best-effort — an API failure falls back to the raw transcript with an informational balloon tip and the normal success sound, while a blank LLM response falls back silently (log only). Only a genuine shutdown cancellation propagates; every other exception falls back regardless of token state.
- Core abstractions: `ILLMProvider`, `ISTTProvider`, `IAudioRecorder`, `IClipboardService`, `ISoundPlayer`, `ITrayNotifier`, `IInputSimulator`
- `OpenAICompatibleLLMProvider` (Services/) with configurable endpoint/model/API
  key, 30 s timeout
- `OperationLock` (Core/) — semaphore guard ensuring single concurrent operation
- `GlobalHotkeyManager` (Core/) — Win32 `RegisterHotKey` P/Invoke with `MOD_NOREPEAT`
- `SettingsForm` (UI/) — hotkey capture, Test Connection, Start with Windows, all NVDA-accessible
- Audio feedback via `SystemSoundPlayer`, balloon notifications via `TrayNotifier`
- `MicrophoneRecorder` (Audio/) — NAudio `WaveInEvent` wrapper with device selection, 16kHz/16-bit/mono WAV, 1-hour auto-stop
- `ElevenLabsSTTProvider` (Services/) — multipart upload to Scribe v2
  (`scribe_v2`), `xi-api-key` header, 5 min timeout, returns transcribed text
- `DictationService` (Core/) — toggle pattern, holds `OperationLock` across record→transcribe→output, releases lock cleanly on `RecordingFaulted`. Dispatches between clipboard and direct insertion via `_settings.DictationOutputMode`.
- Dictation settings tab (UI/) — STT API key (masked), model name, "Test Connection" (uses `SilentWavGenerator` to send a 500 ms silent WAV), microphone dropdown populated from `IAudioRecorder.EnumerateDevices()` with a "(Default)" entry, output-mode radio group (Clipboard / Direct insertion) inside an accessible `GroupBox`, "Clean up dictation with AI" checkbox plus its multiline cleanup prompt box (the prompt box stays enabled when the checkbox is off so it remains reachable in the tab order). `ISTTProvider.TranscribeAsync` has an overload accepting credentials as parameters (mirrors `ILLMProvider.FixTextAsync`), so Test Connection doesn't mutate the live settings singleton
- `DirectInsertionService` (Core/) — orchestrates save→set→paste→delay→restore around `IClipboardService` and `IInputSimulator`. Restore happens in `finally`; non-text or empty original clipboards are not restored (matches the design's stated limitation).
- `Win32InputSimulator` (Core/) — `SendInput` P/Invoke that emits Ctrl down → V down → V up → Ctrl up in a single call.

---
> Source: [ohylli/ai-writing-helper](https://github.com/ohylli/ai-writing-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->

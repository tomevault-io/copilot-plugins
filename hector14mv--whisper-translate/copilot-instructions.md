## whisper-translate

> Voice-to-text translation app using local Whisper for transcription and LLM APIs for translation.

# Whisper Translate

Voice-to-text translation app using local Whisper for transcription and LLM APIs for translation.

## Tech Stack

- **Frontend**: React + TypeScript + Vite + Tailwind CSS
- **Backend**: Tauri 2 (Rust)
- **Transcription**: whisper-rs (local Whisper models)
- **Translation**: OpenAI, Anthropic, Google, or Ollama APIs

## Development

```bash
# Install dependencies
npm install

# Build and run (creates signed .app with microphone permissions)
npm run tauri build

# Copy to Applications
cp -r "src-tauri/target/release/bundle/macos/Whisper Translate.app" /Applications/
```

**Note**: `npm run tauri dev` won't work properly for audio recording because the dev binary lacks microphone entitlements. Always use the built .app for testing audio features.

## Project Structure

```
src/                    # React frontend
  components/           # UI components
  hooks/               # Custom React hooks
  lib/                 # Tauri API bindings
  types/               # TypeScript types

src-tauri/             # Rust backend
  src/
    lib.rs             # Main Tauri commands, settings
    audio.rs           # Audio recording (cpal)
    whisper.rs         # Whisper transcription
    providers.rs       # LLM translation providers
    keychain.rs        # macOS Keychain for API keys
```

## Key Files

- `src-tauri/src/lib.rs` - Settings persistence, main commands
- `src-tauri/src/audio.rs` - Microphone recording to WAV
- `src-tauri/src/whisper.rs` - Local Whisper transcription
- `src/hooks/useTranslation.ts` - Audio processing pipeline
- `src/components/SettingsPanel.tsx` - All settings UI

## Settings Storage

Settings are persisted to `~/.whisper-translate/settings.json`
Whisper models are stored in `~/.whisper-translate/models/`
API keys are stored in macOS Keychain

## Features

- Click-to-record or Push-to-talk recording modes
- Translation toggle (transcription-only mode available)
- Multiple translation providers with pricing display
- Settings persistence across restarts

---
> Source: [hector14mv/whisper-translate](https://github.com/hector14mv/whisper-translate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

## english-practice-streamlit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Progressive Web App (PWA) for English language learning using Text-to-Speech (TTS). Users can create playlists of English-Korean sentence pairs and practice listening with adjustable playback speeds and repeat modes.

## Key Architecture

### Core Components

- **index.html**: Single-page application with two main screens:
  - Upload/playlist selection screen (`uploadCard`)
  - Player screen (`playerCard`)

- **app.js**: State management and application logic
  - Global state: `tracks`, `currentTrack`, `isPlaying`, `isPaused`, `playbackSpeed`, `repeatMode`, `selectedVoice`, `currentProgress`
  - Uses Web Speech API (`SpeechSynthesisUtterance`) for TTS
  - PapaParse library (CDN) for CSV parsing

- **sw.js**: Service Worker for PWA capabilities
  - Cache name: `english-player-v1`
  - Implements cache-first strategy with network fallback

### Data Flow

1. **Input sources**:
   - CSV upload (requires `english` and `korean` columns)
   - Text paste (alternating English/Korean lines)
   - Clipboard paste
   - Sample data

2. **Storage**: localStorage for playlists (key format: `playlist_${timestamp}`)

3. **Playback**:
   - TTS via `window.speechSynthesis`
   - Progress tracking with `onboundary` events for real-time accuracy
   - Pause/resume support (with browser compatibility fallback)
   - Three repeat modes: 'none', 'one', 'all'
   - Voice selection from available system voices

## Development Commands

```bash
# Local development (requires HTTP server for PWA features)
python -m http.server 8000
# or
npx serve

# Access at http://localhost:8000
```

## Deployment

Currently configured for Vercel deployment via `vercel.json` with Service Worker headers.

## Testing TTS Functionality

TTS relies on browser/OS capabilities:
- Test on multiple browsers (Chrome, Safari, Firefox, Edge)
- System language settings affect voice quality
- Use browser DevTools → Console to debug speech synthesis events

## Modifying Playback Behavior

The TTS engine is controlled in app.js:
- `playCurrentTrack()`: Creates and configures `SpeechSynthesisUtterance`
  - Auto-selects best English voice (local service preferred)
  - Sets voice from `selectedVoice` if user has chosen one
  - Uses `onboundary` event to track character-level progress
- `pauseCurrentTrack()`: Attempts `pause()` with fallback to `cancel()`
- `resumeCurrentTrack()`: Resumes paused playback or restarts if cancelled
- Speed control: `utterance.rate` (0.5 to 1.5)
  - Note: Changing speed during playback requires restart (Web Speech API limitation)
- Voice selection: User can choose from available English voices via dropdown
  - Local voices marked with 🔸 indicator
  - Changes take effect immediately if playing
- Event handlers:
  - `onboundary`: Updates progress bar based on character position
  - `onend`: Handles repeat modes and auto-advance
  - `onerror`: Resets playback state on errors

### Web Speech API Limitations

- **No true pause/resume**: Some browsers don't support `pause()`/`resume()` properly, so we fall back to `cancel()` and restart
- **No seeking**: Cannot resume from arbitrary positions; must restart from beginning
- **Progress estimation**: `onboundary` provides character-level tracking, but actual timing depends on TTS engine
- **Voice availability**: Available voices vary by browser and OS

## State Management

All state is managed via global variables in app.js:
- Playlist operations update `tracks` array
- UI updates are imperative (direct DOM manipulation)
- No framework dependencies - vanilla JavaScript

## Service Worker Updates

When modifying cached resources:
1. Update `CACHE_NAME` in sw.js (e.g., `english-player-v2`)
2. Update `urlsToCache` array if adding/removing files
3. Old caches are automatically cleaned up in the `activate` event

## CSV Format

Expected format for imports:
```csv
english,korean
"English sentence here","한국어 번역"
```

Both columns are required and validated in `handleCSVUpload()`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jxcross)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jxcross)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

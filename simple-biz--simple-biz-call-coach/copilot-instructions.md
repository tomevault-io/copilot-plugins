## simple-biz-call-coach

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simple.Biz Call Coach is a Chrome extension that provides real-time AI-powered call coaching for CallTools agents. It captures audio from CallTools.io tabs, transcribes it using Deepgram's WebSocket API, and displays live transcriptions and coaching tips in a side panel.

## Development Commands

### Build and Development
- `npm run dev` - Start Vite development server with hot reload
- `npm run build` - Build extension for production (runs TypeScript compiler + Vite build)
- `npm run preview` - Preview production build

### Loading the Extension
1. Build the extension with `npm run build`
2. Open Chrome and navigate to `chrome://extensions/`
3. Enable "Developer mode"
4. Click "Load unpacked" and select the `dist` folder

## Architecture

### Chrome Extension Components

This is a Manifest V3 Chrome extension with the following components:

1. **Background Service Worker** (`src/background/index.ts`)
   - Central state management hub
   - Coordinates between all components via message passing and port connections
   - Manages tab capture lifecycle and offscreen document
   - Handles state persistence to chrome.storage.local
   - Generates basic coaching tips based on transcription patterns
   - Key state includes: `isRecording`, `isOnCall`, `deepgramStatus`, `transcriptions`, `coachingTips`

2. **Content Script** (`src/content/index.ts`)
   - Injected into `*://*.calltools.io/*` pages
   - Detects active calls using multiple DOM detection methods:
     - Timer element with format `HH:MM:SS`
     - "On a Call" status text
     - Red hangup button visibility
   - Uses port-based connection with background for reliable messaging
   - Implements debouncing (3 confirmations for active, 5 for inactive) to avoid false positives
   - Auto-reconnects if port disconnects

3. **Offscreen Document** (`src/offscreen/index.ts`)
   - Required for tab audio capture (Manifest V3 restriction)
   - Receives audio data via MessagePort from WebRTC Bridge (alternative to tab capture)
   - Manages Deepgram WebSocket connection for live transcription
   - Converts audio to linear16 PCM format for Deepgram
   - Monitors audio levels and broadcasts to UI
   - Handles Deepgram reconnection on connection failure
   - Supports dual audio channels: caller (remote) and agent (local)

3a. **WebRTC Interceptor** (`src/injected/webrtc-interceptor.ts`)
   - Injected into page context (not isolated content script context)
   - Proxies native `RTCPeerConnection` constructor to intercept WebRTC streams
   - Captures both remote (caller) and local (agent mic) audio tracks
   - Hooks `ontrack`, `addEventListener('track')`, and `addTrack` methods
   - Exposes `__getInterceptedStreams()` function to window for content script access
   - Posts messages to window for content script detection (cross-context communication)

3b. **WebRTC Bridge** (`src/content/webrtc-bridge.ts`)
   - Runs in content script context (has chrome API access)
   - Receives intercepted streams from injected script via window messages
   - Creates AudioContext to process both caller and agent audio separately
   - Uses ScriptProcessor nodes to convert Float32 to Int16 PCM
   - Establishes MessageChannel for direct port-based communication with offscreen
   - Streams processed audio chunks to offscreen document with speaker labels ('caller' or 'agent')
   - **Audio Loopback**: Connects processors to `audioContext.destination` for playback

4. **Popup** (`src/popup/`)
   - Browser action popup for quick controls
   - Login/logout functionality (stores user email)
   - Detects if current tab is CallTools.io
   - Shows call status and initiates coaching sessions
   - Polls call state every 2 seconds when on CallTools tab

5. **Side Panel** (`src/sidepanel/`)
   - Main UI for viewing transcriptions and coaching tips
   - Displays live transcriptions with interim/final states
   - Shows Deepgram connection status
   - Audio level visualization
   - Auto-scrolls to latest transcription
   - Persists and syncs state with background via chrome.storage.local

### State Management

- **Zustand Store** (`src/stores/call-store.ts`): React state management for UI components
  - Syncs with chrome.storage.local for persistence
  - Automatically loads persisted state on initialization
  - State includes: `callState`, `audioState`, `deepgramStatus`, `transcriptions`, `coachingTips`, `audioLevel`

- **Settings Store** (`src/stores/settings-store.ts`): User configuration
  - Deepgram API key
  - n8n webhook URL
  - Audio sensitivity
  - Theme preferences

### Message Flow Architecture

The extension uses a hybrid messaging approach:

1. **Port-based connections**: Content script ↔ Background (persistent, auto-reconnect)
2. **Runtime messages**: Background ↔ Popup/SidePanel/Offscreen (broadcast pattern)
3. **Storage events**: State synchronization across components

Key message types:
- `CALL_STARTED` / `CALL_ENDED` - Call lifecycle from content script
- `START_CAPTURE` / `STOP_CAPTURE` - Audio capture control to offscreen
- `GET_WEBRTC_STREAMS` / `STOP_WEBRTC_STREAMING` - WebRTC stream control to bridge
- `WEBRTC_STREAM_PORT` - MessagePort transfer from bridge to offscreen
- `AUDIO_DATA` - Raw audio chunks via MessagePort (with speaker label: 'caller' or 'agent')
- `TRANSCRIPTION_UPDATE` - Live transcription from Deepgram via offscreen
- `DEEPGRAM_STATUS` - Connection status updates
- `STATE_UPDATE` - Full state broadcast to UI components
- `AUDIO_LEVEL_UPDATE` - Real-time audio level monitoring

Window messages (page context → content script):
- `INTERCEPTOR_READY` - WebRTC interceptor initialized
- `REMOTE_AUDIO_TRACK` / `LOCAL_AUDIO_TRACK` - Individual tracks detected
- `AUDIO_TRACKS_READY` - Both caller and agent tracks available

### Critical Implementation Details

1. **Offscreen Document Lifecycle**
   - Must be created before calling `chrome.tabCapture.getMediaStreamId()`
   - Can only be created once - check for existing contexts before creating
   - Path: `src/offscreen/offscreen.html`

2. **WebRTC Audio Capture Flow** (Primary Method)
   1. Content script injects `webrtc-interceptor.ts` into page context at document_start
   2. Interceptor proxies RTCPeerConnection to capture caller (remote) and agent (local) audio tracks
   3. Content script listens for window messages indicating tracks are ready
   4. Background creates offscreen document if not exists
   5. Content script (via `webrtc-bridge.ts`) receives `GET_WEBRTC_STREAMS` message
   6. Bridge retrieves streams via `window.__getInterceptedStreams()`
   7. Bridge creates MessageChannel and sends port1 to offscreen via `WEBRTC_STREAM_PORT`
   8. Bridge processes audio with ScriptProcessor, converting Float32 to Int16
   9. Bridge sends audio chunks to offscreen via MessagePort with speaker labels
   10. Offscreen receives dual-channel audio and sends to Deepgram WebSocket

   **Alternative: Tab Capture Flow** (Fallback)
   1. Background gets stream ID via `chrome.tabCapture.getMediaStreamId({ targetTabId })`
   2. Background sends `START_CAPTURE` message with stream ID to offscreen
   3. Offscreen uses stream ID in `getUserMedia` with `chromeMediaSource: 'tab'`
   4. Offscreen processes and sends single mixed audio stream to Deepgram

3. **Deepgram Configuration** (in `src/offscreen/index.ts`)
   - Model: `nova-2`
   - Encoding: `linear16` (Int16 PCM)
   - Sample rate: `48000` Hz
   - Channels: `1` (mono)
   - Interim results: `true` for live updates
   - Smart formatting: `true` for punctuation

4. **State Persistence Strategy**
   - Background maintains single source of truth in memory
   - Persists to `chrome.storage.local` on every state update
   - UI components load from storage on mount
   - State updates broadcast to all listening components
   - Key storage keys: `callState`, `isOnCall`, `isRecording`, `callStoreState`

5. **Audio Loopback**
   - In WebRTC bridge, audio processors are connected to `audioContext.destination`
   - This allows agents to hear the caller while coaching is active
   - Located in `src/content/webrtc-bridge.ts:163`
   - Essential for normal call operation - agents must hear caller audio

6. **Context Isolation and Script Injection**
   - Content scripts run in isolated world - cannot access page's JavaScript variables
   - WebRTC streams exist in page context, requiring special access pattern:
     1. Inject `webrtc-interceptor.ts` as web_accessible_resource
     2. Interceptor runs in page context, can access RTCPeerConnection
     3. Communication via window.postMessage (cross-context messaging)
     4. Content script retrieves streams via `window.__getInterceptedStreams()`
   - MessagePort used for high-performance audio streaming (avoids postMessage overhead)

## TypeScript Configuration

- Path aliases: `@/*` maps to `./src/*`
- Strict mode enabled with all checks
- React JSX transform
- Chrome extension types via `@types/chrome`

## Build Tool

Uses Vite with `@crxjs/vite-plugin` for Chrome extension development:
- Hot module reload for development
- Automatic manifest processing
- TypeScript compilation
- React fast refresh

## Deepgram Integration Notes

When modifying Deepgram integration:
- API key is stored in chrome.storage.local and passed from background to offscreen
- Offscreen document cannot access chrome.storage directly
- WebSocket URL: `wss://api.deepgram.com/v1/listen`
- Audio must be converted from Float32 to Int16 before sending
- Handle reconnection on WebSocket close (unless clean close code 1000)
- Transcription results have `is_final` flag to distinguish interim vs final

## Debugging

- All components use extensive console logging with emojis for visual parsing:
  - 🚀 Initialization
  - 📞 Call events
  - 🎤 Audio capture
  - 📝 Transcription
  - 🔌 Connections
  - ⚠️ Warnings
  - ❌ Errors
  - ✅ Success

- Check different contexts in Chrome DevTools:
  - Background: `chrome://extensions` → "Inspect views: service worker"
  - Content script: Inspect CallTools.io page
  - Popup: Right-click extension icon → Inspect popup
  - Offscreen: `chrome://extensions` → "Inspect views: offscreen.html"
  - Side panel: Right-click extension icon → Inspect side panel

## Known Constraints

- Only works on `*://*.calltools.io/*` domains
- Requires Deepgram API key for transcription
- Tab capture requires active tab permission
- Service worker may sleep - keep-alive mechanism runs every 20s during calls
- ScriptProcessor (deprecated) is used for audio processing - may need migration to AudioWorklet in future
- WebRTC interceptor must load before CallTools establishes WebRTC connection (uses `run_at: "document_start"`)
- Dual-channel transcription requires Deepgram to support speaker-labeled audio or multichannel processing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Simple-biz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

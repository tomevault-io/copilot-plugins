## demo-azure-avatar-dtt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Real-time voice conversation app with AI-powered lip-synced avatar using Azure VoiceLive SDK. Browser connects via WebSocket to FastAPI backend which proxies to Azure VoiceLive; avatar video streams back via WebRTC.

## Commands

### Backend (Python/FastAPI)

```bash
cd backend
uv sync                                           # Install dependencies (using uv)
uv run uvicorn app.main:app --reload --port 8000  # Run dev server
uv run pytest                                     # Run tests
uv run pytest tests/test_foo.py::test_name        # Run single test
```

### Frontend (Next.js 14)

```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Run dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
```

## Architecture

```
Browser ──WebSocket──► FastAPI Backend ──► Azure VoiceLive
   │                                              │
   │◄════════════ WebRTC Video Stream ════════════│
```

### Backend (`backend/app/`)

- `main.py` - FastAPI WebSocket endpoint at `/ws/voice-avatar`. Accepts client connections, creates `VoiceAvatarSession`, forwards events bidirectionally
- `voice_live.py` - `VoiceAvatarSession` class manages Azure VoiceLive connection. Handles session configuration, audio streaming, WebRTC SDP exchange, and event processing
- `config.py` - Settings class loads Azure credentials and avatar config from environment

### Frontend (`frontend/`)

- `hooks/useVoiceAvatar.ts` - Core hook managing WebSocket connection, WebRTC peer connection, and microphone audio capture. Converts audio to PCM16 base64 for VoiceLive
- `components/VoiceAvatar.tsx` - Main UI component with video player, audio element, transcript panel, and connection controls

### WebSocket Protocol

Client-to-server messages:
- `{type: "audio", data: "<base64 PCM16>"}` - Audio chunks from microphone
- `{type: "avatar.sdp", sdp: "<client SDP>"}` - WebRTC offer for avatar video

Server-to-client messages:
- `{type: "session.ready", ice_servers: [...]}` - Session established, includes ICE servers
- `{type: "avatar.sdp", server_sdp: "..."}` - WebRTC answer
- `{type: "user.speaking.started|stopped"}` - VAD events
- `{type: "transcript", role: "user|assistant", text: "..."}` - Conversation transcript
- `{type: "audio.delta", data: "<base64 PCM16>"}` - Audio chunks for voice-only playback
- `{type: "assistant.speaking.done"}` - Assistant finished speaking
- `{type: "error", message: "...", code: "..."}` - Error events

## Environment Variables

Backend requires `AZURE_VOICELIVE_API_KEY`. See `backend/.env.example` for full list including `AVATAR_CHARACTER`, `AVATAR_STYLE`, `VOICE_NAME`.

Frontend uses `NEXT_PUBLIC_WS_URL` (defaults to `ws://localhost:8000/ws/voice-avatar`).

**Azure Endpoint Configuration**: Two endpoint formats are supported:

1. **AI Foundry resource** (recommended):
   ```
   AZURE_VOICELIVE_ENDPOINT=wss://<instance>.cognitiveservices.azure.com
   ```
   Use the API key from your AI Foundry resource.

2. **Regional Speech Services**:
   ```
   AZURE_VOICELIVE_ENDPOINT=wss://<region>.voice.speech.microsoft.com
   ```
   Requires a Speech Services resource in that region.

See Azure docs for model availability by region: https://learn.microsoft.com/en-us/azure/ai-services/speech-service/regions?tabs=voice-live

## Voice-Only Mode

Avatar streaming requires specific Azure resource configuration. If avatar fails, the app falls back to voice-only mode:
- User speech is transcribed and displayed
- Assistant responses are transcribed and displayed
- Audio is streamed via `audio.delta` WebSocket messages and played in browser

---
> Source: [tonirex/demo-azure-avatar-dtt](https://github.com/tonirex/demo-azure-avatar-dtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

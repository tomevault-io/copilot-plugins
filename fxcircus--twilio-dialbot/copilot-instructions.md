## twilio-dialbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Twilio Dialbot - a browser-based phone dialer that enables outbound PSTN calls with features optimized for navigating IVR systems (like DMV phone menus). It uses WebRTC for browser audio and Twilio for telephony services.

## Key Commands
```bash
# Install dependencies
npm install

# Start the server
npm start

# Note: No tests are configured yet
```

## Architecture & Key Components

### Backend (server.js)
- Express server on port 3000
- JWT token generation at `/token` and `/refresh-token` (tokens expire after 1 hour)
- Creates outbound calls via `/call` endpoint
- Speech recognition endpoints:
  - `/outbound-gather/:phoneNumber` - Initial TwiML with Gather for speech recognition
  - `/transcribe/:callSid` - Real-time partial transcription webhook
  - `/process-speech/:callSid` - Complete speech result processing
- Call state tracking at `/call-state/:callSid`
- Dynamic recording control (only records when human detected)

### Frontend (public/index.html)
- Single-page application with enhanced UI for DMV automation
- Automatic token refresh every 50 minutes
- Business hours enforcement with timezone support
- Real-time call state monitoring and transcription display
- Human detection alerts (audio + browser notification)
- Features: DTMF keypad, auto-redial, audio device selection, call timer

### Automated DMV Call Flow
1. Browser initiates call → Server uses Twilio Gather for speech recognition
2. System detects DMV patterns:
   - "All agents busy" → Auto hang-up and schedule redial
   - "Welcome to DMV" → Says "Agent" twice to navigate menu
3. Monitors for human speech patterns
4. When human detected → Connects browser, starts recording, alerts user
5. Only records human conversations (not IVR)

## Environment Configuration
Required `.env` variables:
```
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_API_KEY_SID=
TWILIO_API_KEY_SECRET=
TWILIO_NUMBER=
SERVER_URL=  # ngrok HTTPS URL (required)
NUMBER_TO_CALL=  # optional
REDIAL_DELAY_MS=    # optional, in milliseconds
```

## Development Requirements
- Twilio account with voice-enabled phone number
- HTTPS required for webhooks and browser mic access
- Browser microphone permissions for device enumeration

## Manual ngrok Setup Required
Due to IT security restrictions:
- ngrok must be installed manually from ngrok.com
- Run `ngrok http 3000` in a separate terminal
- Copy the HTTPS URL to SERVER_URL in your .env file
- Server will validate the URL and display webhook endpoints
- Keep ngrok running while using the app

## Important Notes
- The client-side code uses Twilio Client SDK v1.13 (legacy but stable)
- No Socket.io functionality despite dependency present
- Recording auto-deletion logic: keeps only human-answered calls >2 seconds
- All frontend logic is in index.html - no separate JS files
- Settings (redial delay, selected devices) persist in localStorage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fxcircus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fxcircus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

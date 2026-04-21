## ai-voice-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Client (React frontend)
```bash
cd client
npm run dev       # Dev server at localhost:5173
npm run build     # Production build
npm run lint      # ESLint
npm run test      # Vitest tests
```

### Server (Express backend)
```bash
cd server
npm run dev       # Nodemon dev server (auto-reload)
npm start         # Production server
```

### Deployment
```bash
./deploy.sh       # Build client → install server deps → restart via PM2
```

## Architecture

### System Overview
AI-powered multilingual voice agent platform for CRM. Supports 60+ languages (including Indian languages). Phone calls go through Asterisk/FreeSWITCH → AudioSocket bridge → Express server. Web-based voice chat uses browser MediaRecorder API.

### Frontend (`client/`)
- React 19 + Vite, Tailwind CSS, React Router v7
- API calls go through `client/src/utils/api.js` — an Axios instance that auto-injects JWT tokens and handles 401 logout. In dev, Vite proxies `/api` to `localhost:5001`.
- Auth state lives in `AuthContext` (localStorage-backed JWT). Role-based routing: admins vs. regular users.
- `useVoiceAgent` hook handles browser microphone capture → Deepgram STT → response playback via Sarvam TTS.

### Backend (`server/`)
Entry point: `server/app.js`. Startup sequence: MongoDB → RAG (Pinecone) → Express → WebSocket (call spy) → AudioSocket bridge → cron jobs.

**Key service layers:**

| Layer | Files | Purpose |
|-------|-------|---------|
| Flow Engine | `flowEngine.js`, `stateEngine.js`, `flowGenerator.js` | JSON-defined state machines driving agent conversations |
| AI/LLM | `aiAgent.service.js`, `llmProvider.service.js`, `promptBuilder.js` | Core LLM orchestration with RAG context injection |
| Voice | `deepgram.service.js`, `sarvam.service.js`, `tts.service.js`, `audioProcessor.js` | STT (Deepgram), TTS (Sarvam/ElevenLabs), FFmpeg audio conversion |
| Telephony | `asteriskBridge.service.js`, `asteriskAMI.service.js` | AudioSocket bridge connecting Asterisk calls to server |
| RAG | `ragService.js` | Pinecone vector search; chunks are embedded via OpenAI and retrieved at query time |

**Flow definitions** live in `server/flows/` as JSON. Each flow has steps with `type: "message"` or `type: "input"`, multilingual text blocks (keyed by language code), and transition logic.

### Voice Call Data Flow

**Phone call:**
```
Asterisk/FreeSWITCH → AudioSocket → asteriskBridge.service.js
  → Deepgram STT → aiAgent.service.js (flow engine + RAG + LLM)
  → Sarvam TTS → audio back via AudioSocket
  → Call/Conversation/Lead saved to MongoDB
```

**Web chat:**
```
Browser mic → Deepgram STT → POST /api/chatV5 (streaming)
  → aiAgent.service.js → SSE streamed response
  → Sarvam TTS in browser → <Audio> playback
```

### Key Architectural Patterns
- **Multi-provider abstraction**: `llmProvider.service.js` and `tts.service.js` abstract over multiple providers (OpenAI/XAI for LLM; Sarvam/ElevenLabs for TTS) — swappable via agent config.
- **Flow-based state machine**: Agent behavior is defined in JSON flows, not hardcoded. `stateEngine.js` executes steps and transitions; `slotExtractor.js` extracts structured data from user utterances; `intentClassifier.js` handles routing.
- **Sentence-level streaming**: LLM responses are streamed and split at sentence boundaries so TTS can start playing before the full response is ready (latency optimization in `chatV5` controller).
- **RAG pipeline**: Documents → text extraction (`textExtraction.js`) → chunked → OpenAI embeddings → Pinecone. At query time, top-K chunks are retrieved and injected into the LLM prompt.

### MongoDB Models
`User`, `Agent`, `Call`, `Conversation`, `Campaign`, `Lead`, `PhoneNumber`, `Transaction`, `File`, `ExchangeRate`

### Environment
Server config centralized in `server/config/index.js`. Required env vars include: MongoDB URI, OpenAI key, Deepgram key, Sarvam key, Pinecone key, JWT secret, Razorpay keys.

### Cron Jobs (`server/cron/`)
- Subscription expiry checks
- USD→INR exchange rate updates
- Trial account expiry

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shreyansh0009) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

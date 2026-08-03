## deepremember

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start development environment (both servers):**
```bash
start-dev.bat
```
This opens two terminal windows: backend (port 4004) and client (Vite, port 5173).

**Backend only:**
```bash
cd backend && npm start
```
Note: Backend does not hot-reload. Restart after code changes.

**Client only:**
```bash
cd client && npm run dev
```

**Install all dependencies:**
```bash
npm run install-all
```

**Build for production:**
```bash
cd backend && npm run build
```

## Architecture Overview

This is a full-stack learning application (DeepRemember) with:
- **Backend**: Express.js server (`backend/server.js`)
- **Client**: React 19 + Vite (`client/src/`)

### Backend Structure

```
backend/
├── server.js              # Express entry point
├── config/
│   ├── app.js            # App config (ports, service types, logging)
│   └── database.js       # Database configuration
├── routes/               # API endpoints (/api/srs, /api/chat-templates, etc.)
├── database/access/      # Repository pattern with DatabaseFactory
├── llm/                  # LLM integrations (Groq, Ollama)
├── tts/                  # TTS integrations (Piper, ElevenLabs, Google)
├── stt/                  # STT integrations (LocalWhisper, Groq)
├── filesystem/           # File storage (Node, Google Drive)
└── security/             # Auth middleware and services
```

### Client Structure

```
client/src/
├── main.jsx              # React entry point
├── config/api.js         # API base URL
└── components/
    ├── App.jsx           # Main app with view routing
    ├── security/         # Auth (login, register, context)
    ├── player/           # Audio player, file upload
    ├── chat/             # Chat interface, templates
    ├── ReviewSection/    # SRS flashcard review
    ├── ManageCards/      # Card management
    └── basewords/        # Vocabulary management
```

### Key Design Patterns

**Factory Pattern** for pluggable services:
- `LlmFactory` → Groq or Ollama (via `LLM_PROVIDER` env)
- `TtsFactory` → Piper, ElevenLabs, or Google (via `TTS_TYPE` env)
- `SstFactory` → LocalWhisper or Groq (via `WHISPER_TYPE` env)
- `DatabaseFactory` → SQLite or Supabase (via `DB_TYPE` env)
- `FileSystemFactory` → Node or Google Drive (via `FS_TYPE` env)

**Repository Pattern** for data access:
- `DeepRememberRepository` - Main facade
- `UserConfigRepository`, `WordBaseRepository`, `ChatTemplateRepository`
- All use `IDatabase` interface abstraction

### Database

SQLite (local) or Supabase (cloud). Key tables:
- `cards` - SRS flashcards with FSRS scheduling
- `users`, `labels`, `card_labels`
- `chattemplates`, `user_chattemplates`
- `word_base`, `sentence_analysis_cache`

### Environment Variables

Service selection (in `backend/.env`):
- `DB_TYPE`: supabase | sqlite
- `LLM_PROVIDER`: groq | ollama
- `TTS_TYPE`: piper | elevenlabs | google
- `WHISPER_TYPE`: LocalWhisper | Groq
- `FS_TYPE`: node | googledrive

Logging:
- `DB_LOG=true` - Database query logging
- `LOG_LLM_PROMPTS=true` - LLM prompt logging

## Documentation References

- Database: `backend/SUPABASE_SETUP.md`
- LLM: `backend/llm/LLM.md`
- STT: `backend/stt/README.md`
- TTS: `backend/tts/TTS.md`
- FileSystem: `backend/filesystem/FILESYSTEM.md`

---
> Source: [keremlin/DeepRemember](https://github.com/keremlin/DeepRemember) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->

## multi-provider-langchain

> FastAPI + LangChain multi-provider chat app with real-time Grok voice. Three modes: Chat, Canvas, Voice.

# CLAUDE.md — multi-provider-langchain

## What is this?
FastAPI + LangChain multi-provider chat app with real-time Grok voice. Three modes: Chat, Canvas, Voice.

## Key Commands
```bash
cd ~/dev/multi-provider-langchain/backend && python -m app   # Start server (localhost:8000)
python tests/test_voice_e2e.py                                # Test xAI Voice Agent API
python tests/test_model_perf.py                               # Benchmark model performance
curl http://localhost:8000/api/providers                       # List providers + models
curl http://localhost:8000/api/voice/status                    # Check voice enabled
```

## Environment Variables
```
XAI_API_KEY          # Required for xAI/Grok models + voice
OPENAI_API_KEY       # Required for OpenAI models
ANTHROPIC_API_KEY    # Required for Anthropic models
GROK_VOICE_ENABLED   # Set "true" to enable Voice tab
GROK_VOICE_VOICE     # Default voice: Eve, Ara, Rex, Sal, Leo
```

## File Layout
```
backend/app/
  __main__.py         # Uvicorn entry point (python -m app)
  config.py           # Provider catalog, VoiceConfig, AppSettings
  main.py             # FastAPI routes: /api/chat, /api/voice/ws, /api/providers
  providers.py        # LangChain adapters (OpenAI-compatible + Anthropic)
  voice.py            # TTS synthesis (HTTP POST, one-shot)
  voice_agent.py      # Real-time voice proxy (WebSocket, persistent session)
  context.py          # Conversation summarisation
  store.py            # Filesystem-backed conversation store
  schemas.py          # Pydantic models

frontend/public/
  index.html          # Single-page app shell
  main.js             # App logic, event binding, API calls
  voice.js            # Voice session: mic capture, audio playback, transcript
  audio-processor.js  # AudioWorklet: Float32 → PCM16 at 24kHz
  styles.css          # All styles

tests/
  test_voice_e2e.py   # Standalone xAI Voice Agent API round-trip test
  test_model_perf.py  # Model benchmarking script
```

## Architecture Decisions
- **Voice proxy pattern**: Browser → backend WS → xAI WS. API key never leaves server.
- **AudioWorklet**: Runs in audio thread for low-latency PCM capture (not ScriptProcessorNode).
- **Server VAD**: xAI handles turn detection. No client-side silence detection needed.
- **Vanilla JS**: No build step, no framework. Static files served by FastAPI.
- **LangChain wrappers**: OpenAI-compatible provider handles both OpenAI and xAI.
- **Filesystem store**: Conversations saved as JSON under data/conversations/.

## Conventions
- Python 3.10+, ruff for linting, line length 100
- Pydantic v2 for all schemas
- Async everywhere (FastAPI + asyncio.to_thread for LangChain sync calls)
- No bundler — frontend assets are vanilla JS/CSS served from /static/

## Obsidian Vault
Cross-project knowledge base at: ~/vaults/obsidian_vault/my-claude/
PARA structure: 00_inbox, 01_projects, 02_areas, 03_resources, 04_archives

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SuperJohn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

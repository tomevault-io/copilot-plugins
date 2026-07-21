## ai-watch-buddy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Watch Buddy is a real-time interactive VTuber application that combines video analysis, speech recognition, text-to-speech, and Live2D character animation. The system processes video content and enables users to interact with a virtual character through WebSocket connections.

## Architecture

### Backend (Python FastAPI)
- **FastAPI Server** (`src/ai_watch_buddy/server.py`): WebSocket-based real-time communication server
- **Pipeline System** (`src/ai_watch_buddy/pipeline.py`): Core processing pipeline for video analysis and conversation
- **Agent System** (`src/ai_watch_buddy/agent/`): AI-powered video analysis and action generation
- **ASR/TTS Services** (`src/ai_watch_buddy/asr/`, `src/ai_watch_buddy/tts/`): Speech processing with Fish Audio and Edge TTS
- **Session Management** (`src/ai_watch_buddy/session.py`): User session state and storage
- **Actions System** (`src/ai_watch_buddy/actions.py`): Structured action definitions for character interactions

### Frontend (Next.js + TypeScript)
- **Next.js App** (`frontend/src/app/`): React-based web interface
- **Live2D Integration** (`frontend/WebSDK/`): Complete Live2D Cubism SDK integration for character animation
- **WebSocket Client** (`frontend/src/context/websocket-context.tsx`): Real-time communication with backend
- **Video Player** (`frontend/src/components/video-player-with-controls.tsx`): Integrated video playback with controls
- **Context Management**: React contexts for chat, microphone, settings, and Live2D configuration

## Development Commands

### Backend Development
```bash

# Run with specific environment
uv run python main.py

# Format code
ruff format src/

# Lint code
ruff check src/
```

### Frontend Development
```bash
cd frontend

# Development server
npm run dev

# Build for production
npm run build

# Static export (current configuration)
npm run build  # Creates static export in dist/

# Type checking
npm run type-check

# Linting
npm run lint
```

## Key Dependencies & Environment

### Python Requirements
- Python 3.13+
- FastAPI + Uvicorn for WebSocket server
- Google GenAI for video analysis
- Fish Audio SDK for TTS/ASR
- yt-dlp for video downloading

### Frontend Requirements
- Next.js 15+ with static export configuration
- Live2D Cubism SDK (WebGL rendering)
- React Player for video playback
- Tailwind CSS + shadcn/ui components

### Environment Variables
Required for backend functionality:
- `FISH_AUDIO_API_KEY`: For TTS/ASR services
- `GOOGLE_API_KEY`: For Gemini video analysis
- Additional API keys may be needed based on agent configuration

## Testing

The project includes several test files:
- `test_analyzer.py`: Core analyzer testing
- `test_video_analyzer_basic.py`: Basic video analyzer tests
- `test_video_analyzer_functionality.py`: Advanced functionality tests
- `src/ai_watch_buddy/test_tts_integration.py`: TTS service integration tests

Run tests with: `python -m pytest` or individual test files directly.

## Live2D Integration

The frontend includes a complete Live2D Cubism SDK integration:
- Models stored in `live2d-models/` directory
- WebSDK framework in `frontend/WebSDK/`
- Custom Live2D components in `frontend/src/components/live2d.tsx`
- Expression and animation management through React hooks

## WebSocket Communication

The system uses WebSocket for real-time communication:
- Backend: FastAPI WebSocket endpoints in `server.py`
- Frontend: WebSocket context and connection management
- Message types: Actions, user interactions, system events
- Session-based connection management with automatic reconnection

## File Structure Notes

- `frontend/dist/`: Static export output (should be in .gitignore)
- `video_cache/`: Downloaded video files for processing
- `src/ai_watch_buddy/prompts/`: AI agent prompts and character definitions
- `frontend/libs/`: Static JavaScript libraries for Live2D

---
> Source: [t41372/AI-Watch-Buddy](https://github.com/t41372/AI-Watch-Buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->

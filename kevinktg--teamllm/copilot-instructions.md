## teamllm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a full-stack Live2D avatar application with AI-powered conversational capabilities:

- **Backend** (`backend/`): Node.js Express server with multi-provider AI integration
- **Frontend** (`ui/`): React TypeScript application with Live2D rendering  
- **Root Python** (`main.py`): Simple Python entry point
- **Agents** (`.agents/`): Comprehensive AI CLI tools ecosystem and setup scripts

## Backend Architecture

The backend follows a modular orchestrator pattern:

- `app/main.js` - Express server with `/chat` and `/normalize-asset` endpoints
- `core/orchestrator.js` - Routes requests to appropriate AI providers and personas
- `providers/` - AI provider adapters (claude.js, gemini.js, openai.js)
- `personas/` - Character definitions (aria.js, bricks.js, lemy.js, lordc.js, sol.js)
- `core/emoteTable.js` - Emotion-to-expression mapping
- `core/gcp.js` - Google Cloud Platform integrations for asset processing

## Development Commands

### Backend
```bash
cd backend
npm install
npm run dev    # Development server with --watch
npm start      # Production server
```

### Frontend  
```bash
cd ui
npm install
npm run dev    # Vite dev server with hot reload
npm run build  # Production build
npm run preview # Preview production build
```

### Root Python
```bash
python main.py  # Simple "Hello from team-llm!" output
```

## Required Environment Variables

Backend requires these API keys (see `backend/core/config.js`):
- `GEMINI_API_KEY`
- `OPENAI_API_KEY` 
- `ANTHROPIC_API_KEY`
- `ELEVEN_API_KEY`
- `GOOGLE_CLOUD_PROJECT_ID`
- `GOOGLE_CLOUD_KEY_FILE`
- `GCS_ASSETS_BUCKET`
- `GCS_UPLOADS_BUCKET`

Frontend requires:
- `VITE_API_URL` (defaults to `http://localhost:8080`)

## Live2D Setup

Place required Live2D files in:
- `ui/public/lib/live2dcubismcore.min.js` - Live2D core library
- `ui/public/models/aria/` - ARIA model folder with `aria.model3.json`

## Key Features

- **Multi-Provider AI**: Supports Claude, Gemini, and OpenAI with unified interface
- **Dynamic Personas**: Character-specific system prompts and behaviors
- **Emotion System**: Maps emotional states to avatar expressions
- **Asset Processing**: Google Cloud Vision for image classification
- **Live2D Rendering**: PIXI.js-based 2D avatar display with animations

## Architecture Notes

- Backend uses dynamic imports for lazy-loading providers and personas
- Frontend uses TypeScript with strict type checking
- Both applications designed for containerization (Dockerfile provided)
- Emotion system uses YAML configuration (`backend/emotions.yaml`, `backend/personas.yaml`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinktg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

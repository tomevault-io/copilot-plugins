## demo

> Generates image caption with streaming support.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Image captioning demo web application with streaming AI model support. Users upload images, select an AI model, and receive real-time streaming captions.

**Tech Stack:**
- Frontend: React + TypeScript + Vite
- Backend: Python FastAPI
- AI Models: Anthropic Claude, OpenAI GPT-4 Vision

## Development Commands

### Backend (Python FastAPI)
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### Frontend (React + TypeScript)
```bash
cd frontend
npm install
npm run dev          # Development server (port 5173)
npm run build        # Production build
npm run preview      # Preview production build
```

### Full Stack Development
```bash
# Terminal 1 - Backend
cd backend && source venv/bin/activate && uvicorn main:app --reload --port 8000

# Terminal 2 - Frontend
cd frontend && npm run dev
```

## Architecture

### Request Flow
1. User uploads image in React frontend
2. Frontend sends FormData (image + model_id) to `/api/caption`
3. Backend converts image to base64
4. Backend calls selected AI model with streaming
5. Backend streams response as Server-Sent Events (SSE)
6. Frontend displays caption in real-time

### Backend Structure
- `main.py` - FastAPI app, CORS, endpoints (`/models`, `/caption`)
- `models/anthropic_model.py` - Claude integration with streaming
- `models/openai_model.py` - GPT-4 Vision integration with streaming
- `.env` - API keys (ANTHROPIC_API_KEY, OPENAI_API_KEY)

### Frontend Structure
- `App.tsx` - Main component, state management, API calls
- `components/ImageUpload.tsx` - Drag-and-drop image upload
- `components/ModelSelector.tsx` - Model selection dropdown
- `components/CaptionDisplay.tsx` - Streaming caption display

## Adding New AI Models

1. Create new model file in `backend/models/`:
```python
class NewModel:
    async def generate_caption(self, image_base64: str, media_type: str):
        # Non-streaming implementation
        pass

    async def generate_caption_stream(self, image_base64: str, media_type: str):
        # Streaming implementation - yield "data: {text}\n\n"
        pass
```

2. Register in `backend/main.py`:
```python
from models.new_model import NewModel

models = {
    "new-model-id": NewModel()
}
```

3. Add to `/models` endpoint response in `main.py`

## API Endpoints

### GET /models
Returns available models with metadata:
```json
{
  "models": [
    {
      "id": "claude-3-5-sonnet",
      "name": "Claude 3.5 Sonnet",
      "provider": "Anthropic",
      "supports_streaming": true
    }
  ]
}
```

### POST /caption
Generates image caption with streaming support.

**Request:**
- `file`: Image file (multipart/form-data)
- `model_id`: Model identifier
- `stream`: Boolean (default: true)

**Response:** Server-Sent Events stream with format `data: {text}\n\n`

## Environment Setup

1. Copy `backend/.env.example` to `backend/.env`
2. Add API keys:
   - `ANTHROPIC_API_KEY` - Get from console.anthropic.com
   - `OPENAI_API_KEY` - Get from platform.openai.com

## Key Design Decisions

**Streaming Implementation:**
- Backend uses async generators yielding SSE format
- Frontend uses Fetch API with ReadableStream
- Provides real-time user feedback for long captions

**Base64 Image Encoding:**
- Images converted to base64 before API calls
- Avoids temporary file storage
- Simplifies model integration

**CORS Configuration:**
- Allows localhost:5173 (Vite) and localhost:3000
- Required for frontend-backend communication

**Monorepo Structure:**
- Separate frontend/backend directories
- Root package.json for workspace management
- Independent deployment possible

## Common Tasks

**Test backend endpoint:**
```bash
curl -X POST http://localhost:8000/caption \
  -F "file=@test.jpg" \
  -F "model_id=claude-3-5-sonnet" \
  -F "stream=false"
```

**Check available models:**
```bash
curl http://localhost:8000/models
```

**Frontend API proxy:**
- Vite proxies `/api/*` to `http://localhost:8000`
- Use `/api/models` and `/api/caption` in frontend code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coutHelloworldendl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/coutHelloworldendl)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

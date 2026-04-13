## spinach

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

**Spinach** is a local AI chat application with RAG (Retrieval-Augmented Generation) capabilities. It consists of three main components that must work together:

1. **Frontend** (Next.js 15 + React 19): Chat UI with session management
2. **Backend** (FastAPI): RAG orchestration, vector database management, and LLM proxy
3. **External LLM Server** (llama.cpp): Local LLM inference via OpenAI-compatible API

### Key Architectural Patterns

#### Configuration Flow
- **Frontend**: Uses `app/config.ts` which reads from `.env.local` (environment variables) and `localStorage` (runtime overrides)
  - `config.backendUrl` uses a **getter pattern** to dynamically check localStorage first, then falls back to `NEXT_PUBLIC_BACKEND_URL`
- **Backend**: Uses Pydantic Settings (`backend/config.py`) to load from `.env` with fallback defaults
  - Runtime settings can be updated via `PUT /api/settings` but are **not persisted** to `.env`
- Users can change URLs via the AppInfo UI, which saves frontend URL to localStorage and backend LLM URL via API

#### RAG Pipeline (routes/chat.py)
1. User sends message to `/api/chat/completions` with `use_rag` flag
2. If RAG enabled:
   - Latest user message is embedded using `embedding_model.encode_query()`
   - Vector search queries ChromaDB via `vector_db.query()`
   - Results filtered by similarity threshold (default: 0.5)
   - Context injected into message using `create_rag_prompt()`
3. Modified messages sent to llama.cpp server via `llm_client.chat_completion()`
4. Response streamed back to frontend via SSE (Server-Sent Events)

#### Session Management (app/page.tsx)
- Sessions stored in **localStorage** (client-side only, no backend persistence)
- Each session has unique ID, timestamp, title (from first message), and messages array
- Session list maintained separately in `chat-sessions` localStorage key

#### Lazy Loading Pattern (embeddings.py)
- Embedding model uses lazy loading: initialized on startup but model loaded on first use
- This reduces startup time and memory usage when RAG is not immediately needed

## Development Commands

### Frontend
```bash
npm install                    # Install dependencies
npm run dev                    # Start dev server (http://localhost:3000)
npm run build                  # Production build
npm start                      # Run production server

# Linting and formatting (uses Biome)
npm run lint                   # Fix issues automatically
npm run lint:check             # Check without fixing
npm run format                 # Format code
npm run format:check           # Check formatting
```

### Backend
```bash
cd backend
python -m venv venv            # Create virtual environment
source venv/bin/activate       # Activate (Windows: venv\Scripts\activate)
pip install -r requirements.txt

# Run server
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Or from main.py directly
python main.py
```

### llama.cpp Server
The backend expects llama.cpp server running at `http://localhost:8080/v1` (configurable):
```bash
llama-server \
  --model /path/to/model.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --ctx-size 32768
```

**Important**: llama.cpp does not support runtime model switching via API. To change models, restart the server with a different model file.

## Critical Implementation Details

### Frontend-Backend Communication
- Frontend makes direct fetch calls to backend (configured via `config.backendUrl`)
- Chat uses SSE streaming: listen for `data:` events, stop when `data: [DONE]` received
- Document uploads use multipart/form-data

### ChromaDB Storage
- Persistent storage at `../chroma_data` (relative to backend directory)
- Collection name: "documents" (configurable via `CHROMA_COLLECTION_NAME`)
- Documents are chunked (default: 1000 chars, 200 overlap) before embedding
- Each chunk stored with metadata: `filename`, `chunk_index`, `total_chunks`

### Component Structure
- `app/page.tsx`: Main orchestrator - handles all state and coordinates child components
- `app/components/AppInfo.tsx`: Two-tab interface for model info and settings with edit capabilities
- `app/components/ControlBar.tsx`: Exposes imperative handle via ref for speech recognition control
- Session management entirely in `page.tsx` - no separate service layer

### Environment Variables
Frontend (`.env.local`):
```env
NEXT_PUBLIC_BACKEND_URL=http://localhost:8000
NEXT_PUBLIC_APP_NAME=Spinach
NEXT_PUBLIC_APP_VERSION=0.1.0
```

Backend (`backend/.env`):
```env
LM_STUDIO_BASE_URL=http://localhost:8080/v1
LM_STUDIO_MODEL=/path/to/model.gguf
CHROMA_PERSIST_DIR=../chroma_data
EMBEDDING_MODEL=intfloat/multilingual-e5-base
RAG_TOP_K=3
RAG_SIMILARITY_THRESHOLD=0.5
```

## Common Gotchas

1. **Backend URL Changes**: After editing backend URL in AppInfo, user must reload the page for `config.backendUrl` getter to pick up localStorage changes
2. **Embedding Model Download**: First run downloads ~500MB model - can take several minutes
3. **ChromaDB Path**: Relative paths in `.env` are relative to the backend directory, not project root
4. **Port Conflicts**: Frontend (3000), Backend (8000), and LLM Server (8080) must all be running on different ports
5. **CORS**: Backend CORS origins configured in `.env` (`CORS_ORIGINS`) - must include frontend URL
6. **Biome Ignores**: Backend files are ignored in `biome.json` - don't try to lint Python files

## API Endpoints

### Backend (FastAPI)
- `GET /health` - Health check with ChromaDB status
- `GET /api/llm-info` - Model specs from llama.cpp server
- `GET /api/model-status` - Embedding model loading status
- `GET /api/settings` - Get all backend settings
- `PUT /api/settings` - Update settings at runtime (not persisted)
- `POST /api/chat/completions` - RAG-enabled chat with streaming
- `POST /api/rag/query` - Direct RAG query (no LLM)
- `GET /api/rag/stats` - Vector DB statistics
- `POST /api/documents/upload` - Upload PDF/text files
- `GET /api/documents/list` - List all documents
- `DELETE /api/documents/{id}` - Delete document and its chunks

## Technology Stack Notes

- **Next.js 15**: Using App Router (not Pages Router)
- **React 19**: Client components only (marked with "use client")
- **Tailwind CSS 4**: Custom configuration in `tailwind.config.ts`
- **Biome**: Replaces ESLint + Prettier for faster linting/formatting
- **ChromaDB 0.5.23**: Persistent client mode (not in-memory)
- **Sentence Transformers 3.3.1**: Embedding generation
- **FastAPI 0.115.6**: Async/await patterns throughout

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Takashi-Matsumura)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Takashi-Matsumura)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

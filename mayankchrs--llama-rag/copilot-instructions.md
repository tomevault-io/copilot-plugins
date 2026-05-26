## llama-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start Commands

### 🚀 Start the Complete System (Both Frontend & Backend)

**Option 1: Separate Terminals**
```bash
# Terminal 1 - Backend (NestJS + RAG Pipeline)
cd backend
npm run build && node dist/backend/src/main.js

# Terminal 2 - Frontend (Vue.js + TailwindCSS v4)  
cd frontend
npm run dev
```

**Option 2: Background Processes**
```bash
# Start backend in background
cd backend && npm run build && node dist/backend/src/main.js &

# Start frontend in background
cd frontend && npm run dev &
```

### 📱 Access Points
- **Frontend Application**: http://localhost:5173
- **Backend API Health**: http://localhost:3001/api/health
- **API Documentation**: All endpoints available at http://localhost:3001/api/

### 🔧 Development Commands

#### Backend (NestJS + LlamaIndex.TS + GPT-4o)
```bash
cd backend

# Quick Start (Production Mode)
npm run build && node dist/backend/src/main.js

# Development with Hot Reload
npm run start:dev          # ⚠️ May have module path issues, use above instead

# Build & Test
npm run build              # TypeScript compilation
npm run test               # Unit tests
npm run lint               # ESLint with auto-fix
```

#### Frontend (Vue.js 3 + TailwindCSS v4)
```bash
cd frontend

# Development Server (Vite + HMR)
npm run dev                # Start at http://localhost:5173

# Production Build
npm run build              # Build for production
npm run preview            # Preview production build
```

### ⚠️ Important Setup Notes

#### Required Environment Variables
Create `backend/.env`:
```env
OPENAI_API_KEY=your_openai_api_key_here
PORT=3001
```

#### TailwindCSS v4 Configuration
- ✅ Package.json set to `"type": "module"` (required for ESM)
- ✅ Uses @tailwindcss/vite plugin (no PostCSS needed)
- ⚠️ Avoid `@apply` directive in component styles (known v4 issues)

#### No External Dependencies Required
- ❌ ~~ChromaDB~~ - Using LlamaIndex built-in vector storage  
- ✅ OpenAI API - For GPT-4o LLM and text-embedding-3-small

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) Knowledge Base System** built as a full-stack TypeScript application with a clean separation between frontend, backend, and shared components.

### Core RAG Pipeline Flow
1. **Document Upload** → NestJS DocumentsModule → Multer file handling
2. **Document Processing** → LlamaIndex.TS SimpleDirectoryReader → Text chunking → OpenAI embeddings
3. **Vector Storage** → ChromaDB collection via LlamaIndex ChromaVectorStore
4. **Query Processing** → LlamaIndex QueryEngine → Vector similarity search → OpenAI GPT-3.5-turbo response generation
5. **Response** → Structured response with source citations

### Backend Module Architecture (NestJS)

**DocumentsModule** (`/backend/src/modules/documents/`):
- Handles file uploads with validation (PDF, TXT, CSV, JSON)
- Integrates with LlamaIndex.TS for document processing and vector indexing
- Manages document lifecycle (uploading → processing → ready/failed)
- Stores documents in local filesystem, vectors in ChromaDB

**QueryModule** (`/backend/src/modules/query/`):
- Processes natural language queries using LlamaIndex QueryEngine
- Performs semantic search against ChromaDB vector store
- Generates responses via OpenAI GPT-3.5-turbo with source attribution
- Returns structured responses with similarity scores and metadata

**HealthModule** (`/backend/src/modules/health/`):
- Monitors system health (OpenAI API, ChromaDB connectivity)
- Provides status endpoints for debugging

### Key Technical Integrations

**LlamaIndex.TS v0.11.27** (`llamaindex` + `@llamaindex/openai`):
- Document loading: `SimpleDirectoryReader` for file ingestion
- Vector indexing: `VectorStoreIndex.fromDocuments()`  
- Query processing: `index.asQueryEngine()` with configurable retrieval
- ChromaDB integration: `ChromaVectorStore` for persistent vector storage

**Shared Type System** (`/shared/types.ts`):
- Central TypeScript definitions used across frontend/backend
- Document lifecycle types (`DocumentStatus`, `Document`, `DocumentUploadResponse`)
- Query types (`QueryRequest`, `QueryResponse`, `SourceReference`)
- API response wrappers (`ApiResponse<T>`, `ApiError`)

### Environment Configuration

Backend requires `.env` file:
```
OPENAI_API_KEY=your_key_here
CHROMA_URL=http://localhost:8000
PORT=3001
```

### File Storage Strategy
- **Uploaded Documents**: `/backend/uploads/` (local filesystem)
- **LlamaIndex Storage**: `/backend/storage/` (persistent index storage)
- **Vector Database**: ChromaDB (external service on port 8000)

### API Design Patterns

All endpoints follow consistent patterns:
- Use `ApiResponse<T>` wrapper for consistent response structure
- Implement proper error handling with descriptive messages
- Document processing is asynchronous with status tracking
- CORS enabled for frontend development servers (localhost:3000, localhost:5173)

### Development Workflow

1. Start ChromaDB: `chroma run --host localhost --port 8000`
2. Configure backend `.env` with OpenAI API key
3. Start backend: `cd backend && npm run start:dev`
4. Frontend development ready when implemented
5. Check API health at `/api/health`

### Testing Strategy

The project uses Jest for backend testing:
- Unit tests: `*.spec.ts` files alongside source code
- E2E tests: `/backend/test/` directory
- Coverage reports in `/backend/coverage/`

### Documentation Structure

Critical documentation in `/docs/`:
- `PRD.md`: Product requirements and MVP scope
- `TECHNICAL_ARCHITECTURE.md`: Detailed system design and LlamaIndex configuration
- `TASK_MANAGEMENT.md`: Implementation progress and Sprint tracking

### Type Safety Notes

- Shared types are imported via `/backend/src/shared/types.ts` → `../../../shared/types`
- Frontend will use same shared types for API contracts
- LlamaIndex responses follow specific interface patterns with `sourceNodes` arrays
- All file operations include proper TypeScript typing for Express.Multer.File

---
> Source: [mayankchrs/llama_rag](https://github.com/mayankchrs/llama_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->

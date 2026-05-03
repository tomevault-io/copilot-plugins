## ai-draw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Diagram Hub - An AI-powered diagram creation platform supporting Mermaid, Excalidraw, and Draw.io engines. Users describe diagrams in natural language and AI generates them.

## Development Commands

```bash
# Install dependencies
pnpm install

# Start dev server (frontend + backend together, http://localhost:8787)
pnpm run dev

# Or run separately:
pnpm run dev:frontend   # Vite only (http://localhost:5173)
pnpm run dev:backend    # Wrangler Pages only (http://localhost:8787)

# Build and deploy to Cloudflare Pages
pnpm run pages:deploy

# Other commands
pnpm run build        # TypeScript check + Vite build
pnpm run lint         # ESLint
pnpm run preview      # Preview production build
```

**Note**: 开发时访问 `http://localhost:8787`（wrangler 代理 vite）。

## Architecture

### Monorepo Structure
- **Root**: React frontend (Vite + React 19 + TypeScript)
- **functions/**: Cloudflare Pages Functions (API endpoints)

### Frontend Architecture

**State Management**: Zustand stores in `src/stores/`
- `editorStore.ts` - Current project, canvas content, unsaved changes tracking
- `chatStore.ts` - Chat messages for AI interaction
- `payloadStore.ts` - OpenAI-compatible message payloads

**Data Layer**: Dexie.js (IndexedDB) in `src/lib/db.ts`
- `projects` table - Project metadata with thumbnails
- `versionHistory` table - Content snapshots per project

**Feature Modules** (`src/features/`):
- `engines/` - Drawing engine integrations (mermaid, excalidraw, drawio)
- `chat/` - AI chat panel components
- `editor/` - Canvas and version history UI
- `project/` - Project management

**Services** (`src/services/`):
- `aiService.ts` - Frontend AI client with SSE streaming support
- `projectRepository.ts` / `versionRepository.ts` - IndexedDB CRUD

### Backend Architecture

Cloudflare Pages Functions (`functions/api/`):
- `chat.ts` - AI chat endpoint (OpenAI/Anthropic proxy with streaming)
- `parse-url.ts` - URL content parsing and markdown conversion
- `health.ts` - Health check endpoint
- `_shared/` - Shared utilities (types, CORS, auth, AI providers)

### Key Patterns

**Path Alias**: Use `@/` for imports from `src/` (configured in vite.config.ts and tsconfig)

**Engine Types**: `'mermaid' | 'excalidraw' | 'drawio'` - defined in `src/types/index.ts`

**AI Message Format**: OpenAI-compatible with multimodal support (text + images)

## Environment Setup

Create `.dev.vars` file in root directory for local development:
```env
AI_API_KEY=your-api-key
AI_BASE_URL=https://api.openai.com/v1
AI_PROVIDER=openai
AI_MODEL_ID=gpt-4o-mini
```

For production, configure environment variables in Cloudflare Pages dashboard or use:
```bash
wrangler pages secret put AI_API_KEY
```

---
> Source: [stone-yu/ai-draw](https://github.com/stone-yu/ai-draw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->

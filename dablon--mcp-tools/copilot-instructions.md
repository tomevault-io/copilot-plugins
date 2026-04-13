## mcp-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nodal Aphelion is a monorepo MCP (Model Context Protocol) server platform for media management and AI content generation. It provides tools for media file operations, AI image generation via Stable Diffusion, video creation, presentation generation, and document conversion.

## Build and Development Commands

```bash
# Build all workspaces
npm run build

# Run the MCP server (stdio mode by default, set PORT env for SSE mode)
npm start

# Run all tests with coverage
npm test

# Run a single test file
npx jest tests/media.test.ts

# Run the test agent (requires built server)
npm run test:agent
```

### Docker Compose

```bash
# Start all services
docker-compose up -d

# Start with legacy media-manager
docker-compose --profile legacy up -d

# View logs
docker-compose logs -f mcp-router
```

### Dashboard UI (React/Vite)

```bash
cd dashboard/ui
npm run dev      # Start dev server
npm run build    # Build for production
npm run lint     # Run ESLint
```

## Architecture

### Monorepo Structure

The project uses npm workspaces with this structure:
- `packages/core/` - MCP server factory and transport utilities
- `packages/tools/` - Individual tool packages (media, image-gen, video, presentation, document, researcher, model-manager, crawler)
- `packages/services/` - Microservices for Docker deployment (router, media-service, document-service)
- `apps/mcp-server/` - Main MCP server entry point
- `apps/test-agent/` - Test client for the MCP server
- `dashboard/` - React dashboard with Express backend

### MCP Server (`apps/mcp-server/src/index.ts`)

Uses `@modelcontextprotocol/sdk` with two transport modes:
- **Stdio** (default): For MCP clients via standard input/output
- **SSE** (when `PORT` env set): Express server with `/sse` and `/message` endpoints

### Tool Pattern

Each tool package exports a `register*Tools(server)` function:
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

export function registerMediaTools(server: McpServer) {
    server.tool("list_media", { directory: z.string() }, handleListMedia);
}
```

### Microservices Architecture (Docker)

- **mcp-router** (port 8080): Central orchestrator using gRPC for service communication
- **media-tool** (port 9001): Media file operations
- **document-tool** (port 9002): Document conversion with Ollama AI refinement
- **dashboard** (port 8082): React UI + Express backend
- **image-gen** (port 7860): Stable Diffusion WebUI
- **ollama** (port 11434): LLM for AI refinement
- **scrapper-service** (port 8081): Web scraping and deep crawling service (Rust)

Services communicate via gRPC (proto definition in `proto/tool_service.proto`) and register with the router on startup.

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MEDIA_DIR` | Base directory for media files | `/media` |
| `PORT` | Enables SSE mode if set | - |
| `IMAGE_GEN_API_URL` | Stable Diffusion API URL | `http://localhost:7860` |
| `OLLAMA_API_URL` | Ollama API URL | `http://ollama:11434` |
| `OLLAMA_MODEL` | Model for AI refinement | `gemma3:latest` |
| `OLLAMA_TIMEOUT_MINUTES` | Timeout for AI operations | `5` |
| `SCRAPPER_API_URL` | Scrapper service URL for deep crawling | `http://scrapper-service:8081` |

## Key Tools

- **list_media, get_metadata, move_media, delete_media, upload_image**: File operations
- **generate_image, generate_animation, generate_image_from_image**: Stable Diffusion integration
- **create_slideshow**: FFmpeg-based video creation
- **generate_presentation**: PowerPoint generation with AI backgrounds
- **convert_markdown_to_word, convert_to_word_smart, convert_to_excel_smart**: Document conversion
- **list_models, set_image_model, search_image_models**: Model management
- **deep_crawl**: Deep crawl websites to download media files (images, videos, audio, documents)

## Testing

Tests use Jest with ts-jest ESM preset. Coverage thresholds: 70% branches, 90% functions, 85% lines/statements.

Test files location: `tests/` directory and `packages/**/tests/`.

The jest config maps `@nodal-aphelion/*` imports to the appropriate package sources.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dablon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dablon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

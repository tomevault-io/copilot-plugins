## bytebot-azure

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bytebot is an open-source AI desktop agent - an AI with its own virtual Ubuntu desktop that can complete tasks using real applications. The system consists of four NestJS/Next.js services in a monorepo structure.

## Common Commands

### Full Stack (Docker)
```bash
# Start all services
docker-compose -f docker/docker-compose.yml up -d

# Rebuild and restart
docker-compose -f docker/docker-compose.yml up -d --build
```

### Individual Package Development (from package directory)
```bash
# Install dependencies and build shared types first
cd packages/shared && npm install && npm run build

# Then in any package:
npm install
npm run start:dev     # Development with watch mode
npm run build         # Production build
npm run lint          # ESLint with auto-fix
npm run format        # Prettier formatting
npm test              # Run Jest tests
npm run test:watch    # Tests in watch mode
npm run test:e2e      # End-to-end tests
```

### Database (bytebot-agent)
```bash
npm run prisma:dev    # Run migrations and generate client (development)
npm run prisma:prod   # Deploy migrations and generate client (production)
npx prisma studio     # Open Prisma Studio GUI
```

## Architecture

```
┌─────────────────────┐     ┌─────────────────────┐
│   bytebot-ui        │────▶│   bytebot-agent     │
│   (Next.js :9992)   │     │   (NestJS :9991)    │
│   Web interface     │     │   Task orchestration│
└─────────────────────┘     │   AI providers      │
                            └──────────┬──────────┘
                                       │
                            ┌──────────▼──────────┐
                            │   bytebotd          │
                            │   (NestJS :9990)    │
                            │   Desktop daemon    │
                            │   Ubuntu + XFCE     │
                            │   Mouse/keyboard    │
                            └─────────────────────┘
```

### Packages

- **packages/bytebot-agent** - Main backend: task management, AI provider integration (Anthropic/OpenAI/Google), message persistence. Communicates with bytebotd for desktop control.
- **packages/bytebot-agent-cc** - Alternative agent using Claude Code SDK (`@anthropic-ai/claude-code`)
- **packages/bytebotd** - Desktop daemon: runs inside Ubuntu container, provides computer-use API (mouse, keyboard, screenshots) via nut-js, serves noVNC for remote viewing
- **packages/bytebot-ui** - Next.js frontend with VNC viewer, task creation, real-time updates via Socket.IO
- **packages/shared** - Shared TypeScript types (`MessageContent`, `ComputerAction`) used across packages

### Key Modules (bytebot-agent)
- `AgentModule` - Main AI orchestration and task processing
- `TasksModule` - Task CRUD, REST endpoints, WebSocket gateway
- `AnthropicModule`, `OpenAIModule`, `GoogleModule` - AI provider integrations
- `ProxyModule` - LiteLLM proxy support for 100+ providers

### Key Modules (bytebotd)
- `ComputerUseModule` - Desktop automation (mouse, keyboard, file I/O, screenshots)
- `InputTrackingModule` - User input capture for takeover mode
- `BytebotMcpModule` - Model Context Protocol support

## Database Schema (Prisma)

PostgreSQL with these main models:
- **Task** - Status (PENDING/RUNNING/NEEDS_HELP/NEEDS_REVIEW/COMPLETED/CANCELLED/FAILED), priority, model config
- **Message** - Conversation history, content follows Anthropic's content blocks format (JSON)
- **Summary** - Hierarchical task summaries
- **File** - Uploaded files (base64 encoded)

## Environment Variables

Required:
- `DATABASE_URL` - PostgreSQL connection string
- At least one AI provider key: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, or vLLM configuration

Service URLs (for local dev):
- `BYTEBOT_DESKTOP_BASE_URL=http://localhost:9990`
- `BYTEBOT_AGENT_BASE_URL=http://localhost:9991`
- `BYTEBOT_DESKTOP_VNC_URL=http://localhost:9990/websockify`

vLLM Support (optional):
- `VLLM_BASE_URL` - vLLM server endpoint (e.g., http://localhost:8000)
- `VLLM_API_KEY` - Optional API key for vLLM authentication
- `VLLM_MODEL_NAMES` - Comma-separated list of available models
- `VLLM_CONTEXT_WINDOW` - Context window size (default: 32768)

## Code Patterns

- NestJS modules follow standard structure: `*.module.ts`, `*.service.ts`, `*.controller.ts`, `*.gateway.ts` (WebSocket)
- Shared types must be built before other packages: `npm run build --prefix packages/shared`
- Message content uses Anthropic's format: `[{type: "text", text: "..."}, {type: "image", source: {...}}]`
- Tests use Jest with `*.spec.ts` naming convention

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twelcone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/twelcone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

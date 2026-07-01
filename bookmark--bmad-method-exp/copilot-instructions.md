## bmad-method-exp

> This is a web demo for the BMAD (Business Methodology for Adaptive Design) system, featuring multi-agent collaboration through expansion packs.

# BMAD Web Demo - Project Memory

## Project Overview
This is a web demo for the BMAD (Business Methodology for Adaptive Design) system, featuring multi-agent collaboration through expansion packs.

## Key Components

### Frontend (SvelteKit)
- **Location**: `/apps/web`
- **Port**: 5173
- **Key Features**:
  - Pack selection UI
  - Agent chat interface with WebSocket real-time communication
  - URL routing for packs (`/pack/[packId]`)
  - Markdown rendering for agent responses
  - Typing indicators and status tracking

### Backend (Express + WebSocket)
- **Location**: `/docker/local-server`
- **Port**: 3001
- **Endpoints**:
  - HTTP: `http://localhost:3001`
  - WebSocket: `ws://localhost:3001/ws`
  - API: `http://localhost:3001/api`

### Environment Configuration
- **File**: `/apps/web/.env.development`
```env
VITE_API_URL=http://localhost:3001/api
VITE_WS_URL=ws://localhost:3001/ws
```

## Common Issues and Solutions

### 1. WebSocket Connection Failures
**Issue**: `WebSocket connection to 'ws://localhost:3001/ws' failed`

**Solutions**:
- Ensure server is running: `cd docker/local-server && pnpm run dev`
- If `tsx watch` mode fails, run directly: `npx tsx src/index.ts`
- Check TypeScript errors: `npx tsc --noEmit`
- Verify port is free: `lsof -i :3001`

### 2. Pack Description Parsing
**Issue**: Pack descriptions showing ">-" instead of actual text

**Solution**: Use js-yaml library instead of custom parser
```typescript
import * as yaml from 'js-yaml';
const config = yaml.load(configContent) as ExpansionPackConfig;
```

### 3. Agent YAML Format Variations
The system handles three different agent YAML formats:

1. **Standard format** (most packs)
2. **Full YAML blocks** with `agent:` and `persona:` sections (strategy-consulting)
3. **Root `pack:` key** format (startup-advisor)

### 4. UI Scrolling Issue
**Solution**: Change `overflow-hidden` to `overflow-y-auto` in main layout

## Development Commands

### Start all services
```bash
cd /Users/bookmark/Desktop/vibe-code/project-eclipse/web-demo
pnpm run dev
```

### Start individual services
```bash
# Frontend only
cd apps/web && pnpm run dev

# Backend only
cd docker/local-server && pnpm run dev

# If tsx watch fails, run directly
cd docker/local-server && npx tsx src/index.ts
```

### Testing
```bash
# Test server health
curl http://localhost:3001/health

# Check TypeScript
cd docker/local-server && npx tsc --noEmit
```

## Key Files

### Frontend
- `/apps/web/src/routes/+page.svelte` - Main pack selection page
- `/apps/web/src/routes/pack/[packId]/+page.svelte` - Pack detail/chat page
- `/apps/web/src/lib/components/AgentChat.svelte` - Chat interface
- `/apps/web/src/lib/services/websocket.service.ts` - WebSocket client
- `/apps/web/src/lib/services/pack.service.ts` - Pack API service

### Backend
- `/docker/local-server/src/index.ts` - Express server setup
- `/docker/local-server/src/websocket-handler.ts` - WebSocket message handling
- `/docker/local-server/src/agent-executor.ts` - Agent command execution
- `/docker/local-server/src/api-router.ts` - REST API routes

### Shared
- `/packages/pack-loader/src/loader.ts` - Pack YAML loading
- `/packages/pack-loader/src/registry.ts` - Pack registry management

## Agent Chat Features
- Agent activation with personalized greeting
- `*help` command support
- Natural language processing
- Command parsing and execution
- Markdown formatting in responses
- Real-time typing indicators
- Agent status tracking (idle/thinking/executing)
- Context-aware responses for different agent types

## Notes
- Always ensure the local server is running before testing WebSocket connections
- The system supports multiple concurrent agent conversations
- Agent responses can include visualizations (Mermaid diagrams, ASCII art)
- Each agent has specific expertise and personality traits

---
> Source: [bookmark/BMAD-METHOD-EXP](https://github.com/bookmark/BMAD-METHOD-EXP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

## overture

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Overture is an MCP (Model Context Protocol) server that provides visual plan execution and approval workflows for AI coding agents. It intercepts AI agent planning phases and renders them as interactive visual flowcharts before code is written. Works with Claude Code, Cursor, Cline, GitHub Copilot, and Sixth AI.

## Monorepo Structure

This is an npm workspaces monorepo with two packages:
- `packages/mcp-server` - The MCP server (Node.js/TypeScript)
- `packages/ui` - The React frontend (React Flow + Zustand + Tailwind)

## Build & Development Commands

```bash
# Install dependencies (from root)
npm install

# Development - runs both server and UI with hot reload
npm run dev

# Build all packages for production
npm run build

# Development - individual packages
npm run dev:server    # MCP server only (tsx watch)
npm run dev:ui        # UI only (vite dev server)

# Build - individual packages
npm run build:server  # Outputs to packages/mcp-server/dist/
npm run build:ui      # Outputs to packages/mcp-server/ui-dist/

# Production start
npm start             # Runs built MCP server

# Clean all build artifacts
npm run clean
```

### MCP Server Package Commands

```bash
cd packages/mcp-server
npm run dev           # tsx watch src/index.ts
npm run build         # tsup to dist/
npm run lint          # eslint src/**/*.ts
npm run start         # node dist/index.js
```

### UI Package Commands

```bash
cd packages/ui
npm run dev           # vite dev server
npm run build         # tsc && vite build
npm run preview       # vite preview
```

## Architecture

### MCP Server (`packages/mcp-server/src/`)

**Entry Points:**
- `index.ts` - MCP server initialization using `@modelcontextprotocol/sdk`, starts HTTP (3031) and WebSocket (3030) servers
- `cli.ts` - CLI entry point for `npx overture-mcp`

**Core Modules:**
- `tools/handlers.ts` - Implements all 11 MCP tools (`submit_plan`, `get_approval`, `update_node_status`, etc.)
- `store/plan-store.ts` - `MultiProjectPlanStore` manages plans across multiple projects with Promise-based approval flow
- `websocket/ws-server.ts` - `WebSocketManager` handles client connections, project subscriptions, and relay mode
- `http/server.ts` - Express server serving UI and API endpoints (`/api/test-plan`, `/api/mcp-marketplace`)
- `parser/xml-parser.ts` - SAX-based `StreamingXMLParser` for parsing plan XML into nodes/edges
- `parser/output-parser.ts` - Parses structured output from agent execution
- `storage/history-storage.ts` - Persists plans to `~/.overture/history.json`
- `types.ts` - Core types: `PlanNode`, `PlanEdge`, `Plan`, `ProjectContext`, `WSMessage`
- `utils/plan-diff.ts` - Computes differences between plan versions

**Key Patterns:**
- Projects identified by `projectId` (SHA256 hash of workspace path, first 12 chars)
- Branch points detected from graph structure (nodes with multiple outgoing edges)
- WebSocket relay mode allows multiple MCP server instances to share one UI
- Plans auto-saved every 3 seconds to `~/.overture/history.json`

**Environment Variables:**
- `OVERTURE_HTTP_PORT` (default: 3031)
- `OVERTURE_WS_PORT` (default: 3030)
- `OVERTURE_AUTO_OPEN` (default: true)

### UI (`packages/ui/src/`)

**Entry Points:**
- `main.tsx` - React app initialization
- `App.tsx` - Main application component

**State Management:**
- `stores/plan-store.ts` - Zustand store for plan state, node status, selections
- `stores/multi-project-store.ts` - Manages multiple project contexts and tab switching

**Key Components:**
- `components/Canvas/PlanCanvas.tsx` - React Flow canvas with nodes and edges
- `components/Canvas/TaskNode.tsx` - Task node rendering with status indicators
- `components/Canvas/InsertableEdge.tsx` - Custom edge with insert capability
- `components/Panel/NodeDetailPanel.tsx` - Side panel showing node details, fields, branches
- `components/Panel/StructuredOutputView.tsx` - Renders execution output with expandable sections
- `components/Controls/ApproveButton.tsx` - Smart approval button with status awareness
- `components/Controls/RequirementsChecklist.tsx` - Shows pending fields and branch selections
- `components/Modals/BranchSelectionModal.tsx` - Side-by-side branch comparison
- `components/Modals/McpMarketplaceModal.tsx` - MCP server marketplace browser

**WebSocket Hook:**
- `hooks/useWebSocket.ts` - Manages WebSocket connection and message handling

**Layout Hook:**
- `hooks/useAutoLayout.ts` - Dagre-based automatic node positioning

### Agent Prompts (`prompts/`)

Contains usage instructions for different AI agents loaded by `get_usage_instructions` tool:
- `claude-code.md`, `cline.md`, `cursor.md`, `sixth.md`, `gh_copilot.md`

## MCP Tools Reference

| Tool | Purpose |
|------|---------|
| `submit_plan` | Submit plan as XML |
| `get_approval` | Block until user approves |
| `update_node_status` | Update node status + output during execution |
| `plan_completed` | Mark plan as completed |
| `plan_failed` | Mark plan as failed |
| `check_rerun` | Check if user requested node re-run |
| `check_pause` | Check if user paused execution |
| `get_resume_info` | Get context for resuming paused plan |
| `request_plan_update` | Request incremental plan modifications |
| `create_new_plan` | Signal new plan creation |
| `get_usage_instructions` | Get agent-specific instructions |

## Tech Stack

- **MCP Server:** Node.js, TypeScript, Express, ws, @modelcontextprotocol/sdk, sax, zod
- **UI:** React 18, @xyflow/react (React Flow), Zustand, Framer Motion, Tailwind CSS, Vite
- **Build:** tsup (server), Vite (UI)
- **Layout:** dagre (automatic graph positioning)


## BUILD INSTRUCTIONS NECESSARY TO TEST WHAT YOU HAVE DONE
- **Rebuilding UI Files** Always rebubild the files being served, so that the most recent and updated code is what is being redered or tested

---
> Source: [SixHq/Overture](https://github.com/SixHq/Overture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->

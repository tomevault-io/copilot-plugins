## claude-code-agents-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

agents-ui is a Nuxt 3-based visual dashboard for managing Claude Code agents, commands, skills, workflows, and plugins. It provides a GUI layer on top of the `~/.claude` directory, allowing users to create, edit, and organize their Claude Code configuration without touching markdown files directly.

## Development Commands

```bash
# Development
bun run dev          # Start dev server at http://localhost:3030
npm run dev          # Alternative with npm

# Build & Production
bun run build        # Build for production
bun run preview      # Preview production build

# Type Checking
bun run typecheck    # Run TypeScript type checking
```

## Architecture

### Frontend (Nuxt 3 + Vue 3)

**Pages** (`app/pages/`):
- `/agents` - List and manage agents
- `/agents/[slug]` - Edit individual agent
- `/commands` - List and manage commands
- `/skills` - List and manage skills
- `/workflows` - Visual workflow builder
- `/graph` - Relationship visualization
- `/explore` - Browse templates and marketplace
- `/cli` - Full terminal emulator with context monitoring
- `/settings` - Global settings

**Composables** (`app/composables/`):
The app uses a centralized CRUD pattern via `useCrud.ts` which provides standard `fetchAll`, `fetchOne`, `create`, `update`, `remove` operations. Domain-specific composables wrap this:
- `useAgents.ts` - Agent CRUD operations
- `useCommands.ts` - Command CRUD operations
- `useSkills.ts` - Skill CRUD operations
- `useWorkflows.ts` - Workflow CRUD operations
- `useStudioChat.ts` - Agent Studio chat with SSE streaming
- `useGithubImports.ts` - Import skills from GitHub repos
- `useMarketplace.ts` - Browse and install plugins from marketplace
- `useTerminal.ts` - Terminal emulator with xterm.js integration and WebSocket streaming
- `useContextMonitor.ts` - Real-time context metrics tracking (tokens, cost, files, tools)
- `useCliExecution.ts` - CLI session management with agent-aware and standalone modes

**Chat/Studio System**:
The Agent Studio (`/agents/[slug]` page with test panel) uses SSE (Server-Sent Events) for streaming responses. The backend (`server/api/chat.post.ts`) uses the `@anthropic-ai/claude-agent-sdk` to query agents and stream results back as `text_delta`, `thinking_delta`, `tool_progress` events.

### Backend (Nuxt Server API)

**File System Layer** (`server/utils/`):
- `claudeDir.ts` - Resolves `~/.claude` path (respects `CLAUDE_DIR` env var)
- `frontmatter.ts` - Parse/serialize YAML frontmatter + markdown body
- `relationships.ts` - Extract relationships between agents/commands/skills by scanning frontmatter (`agent:` field) and body text for references
- `github.ts` - Clone, scan, and import skills from GitHub repos
- `marketplace.ts` - Fetch and install plugins from marketplace sources

**API Routes** (`server/api/`):
All CRUD routes follow REST conventions:
- `GET /api/agents` - List all
- `GET /api/agents/[slug]` - Get one
- `POST /api/agents` - Create
- `PUT /api/agents/[slug]` - Update
- `DELETE /api/agents/[slug]` - Delete

Special endpoints:
- `POST /api/chat` - SSE endpoint for Agent Studio, uses `@anthropic-ai/claude-agent-sdk` to execute agents with streaming
- `GET /api/relationships` - Build graph data by extracting relationships
- `GET /api/agents/[slug]/skills` - List skills assigned to an agent
- `POST /api/github/import` - Import skills from GitHub
- `POST /api/marketplace/install` - Install plugin from marketplace

### Data Model

**Agents** (`~/.claude/agents/*.md`):
```yaml
---
name: Agent Name
description: What the agent does
model: sonnet | opus | haiku
color: "#hex"
memory: user | project | none
---

Agent instructions go here...
```

**Commands** (`~/.claude/commands/**/*.md`):
```yaml
---
name: command-name
description: What the command does
argument-hint: "[optional args]"
allowed-tools: [Read, Write, Edit]
agent: agent-slug  # optional, link to agent
---

Command prompt goes here...
```

**Skills** (`~/.claude/skills/[name]/SKILL.md`):
```yaml
---
name: skill-name
description: What the skill does
context: when | always
agent: agent-slug  # optional, link to agent
---

Skill prompt goes here...
```

**Workflows** (`~/.claude/workflows/*.json`):
```json
{
  "name": "Workflow Name",
  "description": "Description",
  "steps": [
    { "id": "step-1", "agentSlug": "agent-name", "label": "Step label" }
  ],
  "createdAt": "ISO timestamp"
}
```

### Key Patterns

**Relationship Detection**:
The system automatically detects relationships between agents/commands/skills by:
1. Checking `agent:` frontmatter field (links command/skill → agent)
2. Scanning body text for `subagent_type: "agent-name"` patterns
3. Scanning for `/command-name` references in agent bodies
4. Matching direct mentions of agent slugs in text

**Studio Chat System**:
- Frontend uses `useStudioChat()` which streams SSE events
- Backend uses `@anthropic-ai/claude-agent-sdk`'s `query()` function
- System prompt is either the default "Agent Manager" prompt or the selected agent's instructions
- Session IDs enable conversation continuation across multiple messages
- Tool progress and thinking blocks are streamed incrementally

**GitHub Import Flow**:
1. User provides GitHub URL
2. Backend clones repo to temp dir
3. Scans for SKILL.md files (checks frontmatter for `type: skill`)
4. User selects which skills to import
5. Skills are copied to `~/.claude/skills/[name]/`
6. Import metadata stored in `~/.claude/.imports.json` for update tracking

**CLI Terminal System** (`/cli` page):
The CLI provides a full terminal emulator with real-time context monitoring for tracking token usage, costs, file changes, and tool executions.

### Architecture

**Frontend Components** (`app/components/cli/`):
- `Terminal.vue` - Xterm.js terminal with WebSocket streaming
- `ContextPanel.vue` - Tabbed context monitoring panel (Metrics, Files, Tools, History)
- `MetricsCard.vue` - Token usage, context window, and cost tracking with progress bars
- `FileTree.vue` - Real-time file system changes with type indicators
- `ToolTimeline.vue` - Tool execution timeline with status and duration
- `SessionHistory.vue` - Active and historical session list

**Backend Infrastructure** (`server/`):
- `server/utils/cliSession.ts` - PTY session management with node-pty
- `server/utils/contextMonitor.ts` - Token parsing, cost calculation, file watching with chokidar
- `server/api/cli/ws.ts` - WebSocket handler for bidirectional terminal I/O
- `server/api/cli/sessions/` - REST endpoints for session CRUD

### WebSocket Protocol

**Client → Server Messages**:
```typescript
{ type: 'execute', agentSlug?, workingDir?, cols?, rows? }  // Start new session
{ type: 'input', sessionId, data }                          // Send terminal input
{ type: 'resize', sessionId, cols, rows }                   // Resize terminal
{ type: 'kill', sessionId }                                 // Terminate session
```

**Server → Client Events**:
```typescript
{ type: 'session', sessionId }                    // Session created
{ type: 'output', data }                          // Terminal output
{ type: 'context_update', metrics }               // Full context metrics
{ type: 'token_update', tokens }                  // Incremental token update
{ type: 'file_change', change }                   // File system change
{ type: 'tool_call', tool }                       // Tool execution event
{ type: 'error', error }                          // Error message
{ type: 'exit', exitCode }                        // Session terminated
```

### Session Management

**Session Storage**:
- Active sessions: In-memory Map with PTY instances, metadata, file watchers
- Session history: Saved to `~/.claude/cli-history/{sessionId}.json`
- Auto-cleanup: Sessions terminate after 30 minutes of inactivity
- Output buffering: Last 10,000 lines stored per session

**Context Monitoring**:
- Token parsing: Extracts token counts from Claude CLI output patterns
- Cost calculation: Real-time pricing based on model (Sonnet/Opus/Haiku)
- File watching: Chokidar monitors working directory for creates/modifies/deletes
- Tool tracking: Parses tool calls from output with status and elapsed time

### Dual Mode Operation

**Agent-Aware Mode**:
- User selects an agent from dropdown
- Agent's instructions used as system prompt
- Agent's model and settings applied
- Session linked to agent in history

**Standalone Mode**:
- No agent selection required
- Default shell environment
- Independent session tracking
- Quick terminal access

### Key Features

1. **Full Terminal Emulation**: Uses xterm.js + node-pty for complete shell experience
2. **Real-time Metrics**: Live token counting, cost tracking, context window visualization
3. **File System Monitoring**: Automatic detection of file changes during execution
4. **Tool Timeline**: Visual timeline of all tool calls with status and duration
5. **Session Persistence**: Sessions saved to disk with full output history
6. **Responsive UI**: 60/40 split view (terminal/context) with collapsible panels

---

**Claude Code Chat Mode** (`/cli` page - Chat tab):
The Chat mode provides a web-based chat interface that directly integrates with the Claude Code SDK for conversational interactions.

### Architecture

**Frontend Components** (`app/components/cli/chat/`):
- `ChatInterface.vue` - Main chat container with message list and input
- `ChatMessages.vue` - Message list renderer
- `MessageItem.vue` - Individual message component with markdown, tool rendering, thinking blocks
- `ChatInput.vue` - Textarea input with auto-resize and keyboard shortcuts

**Backend Infrastructure**:
- `server/api/chat/ws.ts` - WebSocket handler for bidirectional chat communication
- `server/utils/claudeSdk.ts` - Direct integration with `@anthropic-ai/claude-agent-sdk`
- `server/utils/messageNormalizer.ts` - Converts SDK events to unified NormalizedMessage format
- `server/utils/chatSessionStorage.ts` - JSONL-based session persistence
- `server/api/chat/sessions/` - REST endpoints for session CRUD

**Frontend Composables**:
- `useChatSessions.ts` - Session state management with message store
- `useWebSocketChat.ts` - WebSocket client with auto-reconnect and streaming handling

### Message System

**NormalizedMessage Format**:
All SDK events are normalized to a unified message type with different `kind` values:
- `text` - User or assistant text messages
- `thinking` - Extended thinking blocks (collapsible)
- `tool_use` - Tool being called with parameters
- `tool_result` - Tool execution result
- `stream_delta` - Streaming text chunks (accumulated in real-time)
- `stream_end` - Stream complete signal
- `complete` - Query finished
- `error` - Error occurred

**Message Flow**:
```
User types message
  ↓
WebSocket: { type: 'start', message, sessionId, agentSlug }
  ↓
Backend calls query() from SDK
  ↓
SDK events normalized to NormalizedMessage
  ↓
Sent via WebSocket to client
  ↓
Frontend accumulates streaming text
  ↓
Display in ChatMessages component
  ↓
Save to JSONL file (~/.claude/chat-sessions/{sessionId}.jsonl)
```

### Session Storage

**Format**: JSONL (JSON Lines)
- One message per line
- Append-only for performance
- Stored in `~/.claude/chat-sessions/{sessionId}.jsonl`

**Pagination**:
- Initial load: Last 50 messages
- "Load more" button fetches older messages
- Pagination managed by backend

### Streaming Implementation

**Buffered Updates**:
- Stream deltas accumulated in `streamingText` ref
- Real-time display with cursor animation
- Finalized to permanent message on `stream_end`

**Performance**:
- No batching (immediate display)
- Single streaming message at a time
- Auto-scroll to bottom on new messages

### UI Features

**Message Display**:
- User messages: Right-aligned, blue background
- Assistant messages: Left-aligned, markdown rendering
- Tool use: Collapsible sections with parameters/results
- Thinking blocks: Collapsed by default, expandable
- Errors: Red highlighted banners

**Input Composer**:
- Auto-resizing textarea (max 200px height)
- Enter to send, Shift+Enter for newline
- Character counter
- Disabled during streaming

**Status Indicators**:
- Connected/Disconnected badge
- "Generating..." indicator during streaming
- Animated thinking dots before text appears

### Agent Integration

**Agent-Aware Mode**:
- Agent selector in top bar
- Agent instructions passed as `systemPrompt` to SDK
- Agent slug saved with session
- Sessions linked to agents in history

**Standalone Mode**:
- No agent selection required
- Default SDK behavior
- Quicker access for ad-hoc queries

### Mode Toggle

The `/cli` page has tabs to switch between:
- **Terminal** - Full PTY terminal emulator
- **Chat** - Conversational Claude Code interface

Both modes share the same agent selector and working directory settings.

## Testing

When adding new features:
- Test file CRUD operations by checking files are created/updated/deleted in `~/.claude/`
- Test relationship detection by creating agents/commands with cross-references
- Test Studio chat by verifying SSE events stream correctly
- Test GitHub imports with real repos (e.g., public skill repos)
- Test CLI terminal by verifying PTY spawning, WebSocket streaming, and context monitoring
- Test session management by creating/terminating sessions and checking `~/.claude/cli-history/`

## Environment Variables

```bash
CLAUDE_DIR="~/.claude"  # Override default Claude config directory
```

## Component Organization

Components in `app/components/` are auto-imported with special prefixing:
- `chat/*` - Chat UI components (no prefix)
- `studio/*` - Agent Studio components (no prefix)
- `cli/*` - CLI terminal components (no prefix)
- Everything else - Standard component naming

## Type Definitions

All TypeScript types are centralized in `app/types/index.ts`. Key types:
- `Agent`, `Command`, `Skill`, `Workflow` - Core entities
- `Relationship` - Links between entities
- `ChatMessage`, `StreamActivity` - Studio chat
- `GithubImport`, `Plugin` - External integrations
- `CliSession`, `ContextMetrics`, `TokenUsage`, `FileChange`, `ToolCall` - CLI terminal and monitoring
- `CliWebSocketMessage`, `CliWebSocketEvent` - Terminal WebSocket types
- `NormalizedMessage`, `ChatSession`, `ChatSessionSummary` - Chat mode types
- `ChatWebSocketMessage`, `ChatWebSocketEvent` - Chat WebSocket types

## Model Registry Design

All model-related data is centralized in **two canonical files**. Never inline model colors, labels, pricing, option lists, or model string comparisons elsewhere.

### Frontend — `app/utils/models.ts`

Single source of truth for all UI-facing model metadata:

```typescript
// ✅ Use these in all .vue files and app/utils/
import {
  MODEL,               // { OPUS, SONNET, HAIKU } — named constants for comparisons/defaults
  MODEL_IDS,           // ['opus', 'sonnet', 'haiku'] — canonical list for iteration
  MODEL_META,          // Full per-model metadata (label, description, colors, etc.)
  MODEL_OPTIONS,       // Options array for picker UIs (includes "Default" entry)
  MODEL_OPTIONS_COMPACT, // Compact options for toggle-button pickers
  MODEL_OPTIONS_CHAT,  // Options with { value, label, description } for chat selectors
  DEFAULT_MODEL,       // 'sonnet' — default when no model is specified
  getModelLabel,       // Human-readable label
  getModelTagline,     // One-liner ("Balanced", "Most capable", ...)
  getModelColor,       // Hex color for charts/bars
  getModelBadgeClasses, // Tailwind bg+text classes for badges
  getModelBadgeStyle,  // Inline style object (use in templates with dynamic binding)
} from '~/utils/models'
```

**Comparisons and defaults — always use constants, never raw strings:**

```typescript
// ✅ Correct
import { MODEL, DEFAULT_MODEL } from '~/utils/models'
const selectedModel = ref(DEFAULT_MODEL)           // not ref('sonnet')
if (model === MODEL.SONNET) { ... }               // not if (model === 'sonnet')
frontmatter.model = MODEL.OPUS                    // not frontmatter.model = 'opus'

// ✅ Iterating all models
import { MODEL_IDS } from '~/utils/models'
MODEL_IDS.map(id => ({ value: id, label: MODEL_META[id].label }))

// ❌ Never
const selectedModel = ref('sonnet')
if (model === 'opus') { ... }
```

Each `ModelMeta` entry contains: `label`, `tagline`, `description`, `badgeBg`, `badgeText`, `color`, `contextWindow`.

**Adding a new model**: add one entry to `MODEL`, `MODEL_IDS`, and `MODEL_META`. All downstream UI automatically picks it up.

### Server — `server/utils/models.ts`

Server-side mirror with pricing and API model IDs:

```typescript
import {
  MODEL_ALIAS_KEY,      // { OPUS, SONNET, HAIKU } — named alias key constants
  DEFAULT_MODEL_ALIAS,  // 'sonnet' — server-side default
  SERVER_MODEL_META,    // Full pricing + context window per model id
  MODEL_ALIAS,          // 'sonnet' → 'claude-sonnet-4' mapping (full ids)
  getModelPricing,      // Pricing for cost calculation
  getModelContextWindow, // Context window for utilization tracking
  resolveModelMeta,     // Resolve alias OR full id → ServerModelMeta
} from './models'       // (relative import from server/utils/)
```

```typescript
// ✅ Correct (server-side)
import { MODEL_ALIAS_KEY } from '../models'
models: Object.values(MODEL_ALIAS_KEY)   // not ['sonnet', 'opus', 'haiku']
if (model === MODEL_ALIAS_KEY.SONNET)    // not if (model === 'sonnet')
```

**Updating pricing**: edit `SERVER_MODEL_META` in `server/utils/models.ts` only.

### Design Principles

1. **One edit = one place**: Adding a model → edit `MODEL`/`MODEL_IDS`/`MODEL_META` (frontend) and `MODEL_ALIAS_KEY`/`MODEL_ALIAS`/`SERVER_MODEL_META` (server). All consumers update automatically.
2. **No raw string literals in logic**: String literals (`'sonnet'`, `'opus'`) live only in `models.ts` definitions. Every comparison and default elsewhere is a `MODEL.X` constant.
3. **No inline model data**: No hardcoded `rgba()` per model in templates. No `{ opus: '...', sonnet: '...', haiku: '...' }` spread across components.
4. **Frontend/server split**: App utils cannot be imported server-side (different module context). Each layer has its own registry file.
5. **Backwards compat**: Legacy helpers (e.g., `getFriendlyModelName` in `terminology.ts`) are kept with `@deprecated` JSDoc and delegate to the new helpers.

---
> Source: [Ngxba/claude-code-agents-ui](https://github.com/Ngxba/claude-code-agents-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->

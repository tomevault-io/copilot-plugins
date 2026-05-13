## pocket-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pocket Server is a Node.js + Hono WebSocket server that provides AI agent capabilities, file system operations, terminal management, and background agent orchestration. Built with TypeScript and ESM modules, it serves as the backend for the Pocket Agent mobile app.

## Architecture

### Core Design Principles
- **Event-driven mailbox pattern** - Each session has its own AgentRunner with strict ordering
- **Server-authoritative storage** - All conversation data persisted to `data/sessions/`
- **Streaming-first** - Optimized for real-time WebSocket streaming with frame aggregation
- **Module isolation** - Clean separation between agent, auth, file system, and terminal modules

### Module Structure

```
src/
├── agent/                       # AI agent orchestration
│   ├── anthropic/              # Anthropic Claude implementation
│   │   ├── anthropic.ts        # Main agent logic and streaming
│   │   ├── prompt.ts           # System prompts and context
│   │   ├── streaming.ts        # Stream event processing
│   │   ├── title.ts            # Title generation
│   │   ├── tools/              # Tool implementations
│   │   │   ├── bash.ts         # Terminal command execution
│   │   │   ├── editor.ts       # File editing operations
│   │   │   └── web-search.ts   # Web search capability
│   │   └── types.ts            # Anthropic API types
│   ├── store/                  # Session persistence
│   │   └── session-store-fs.ts # File-based session storage
│   └── index.ts                # Agent module registration
│
├── auth/                       # Authentication & authorization
│   ├── device-registry.ts     # Device management
│   ├── middleware.ts           # Auth verification middleware
│   ├── pairing.ts             # Local PIN pairing window
│   ├── routes.ts              # Auth HTTP endpoints
│   └── token.ts               # Token generation/validation
│
├── background-agent/           # Cloud agent integrations
│   └── cursor/                # Cursor IDE integration
│       ├── client.ts          # Cursor API client
│       ├── github.ts          # GitHub integration
│       ├── routes.ts          # Cursor HTTP endpoints
│       ├── tracker.ts         # Agent status tracking
│       └── types.ts           # Cursor-specific types
│
├── file-system/               # File operations
│   ├── handlers.ts           # WebSocket handlers
│   ├── service.ts            # Core file operations
│   ├── telescope-search.ts   # Fast file search
│   ├── terminal.ts           # Terminal command execution
│   └── types.ts              # File system types
│
├── notifications/             # Push notifications
│   └── index.ts              # Expo push integration
│
├── server/                    # Core server infrastructure
│   ├── router.ts             # Custom Hono router adapter
│   └── websocket.ts          # WebSocket management
│
├── shared/                    # Shared utilities
│   ├── logger.ts             # Structured logging
│   ├── paths.ts              # Path resolution utilities
│   ├── public-url.ts         # Public URL management
│   └── types/
│       └── api.ts            # Shared API types
│
├── terminal/                  # Terminal management
│   └── terminal-manager.ts   # PTY session management
│
├── tunnel/                    # Remote access
│   └── cloudflare.ts         # Cloudflare quick tunnel
│
├── cli.ts                     # CLI entry point
└── index.ts                   # Server entry point
```

## Development Commands

```bash
# Development with hot reload
npm run dev              # tsx watch mode on port 3000

# Production build
npm run build            # Bundle with esbuild to dist/
npm start                # Run production server

# Testing
npm test                 # Run all tests
npm test -- src/foo.test.ts  # Run specific test

# Linting (gates releases)
npm run lint             # Biome linting with strict rules

# CLI commands
npm run run start              # Start server
npm run run start --remote     # Start with Cloudflare tunnel
npm run run pair               # Open pairing window
npm run run stop               # Stop server
npm run run -- --help          # Show CLI usage
npm run run update       # Update to latest version
```

## Key APIs and Protocols

### WebSocket Protocol

All WebSocket messages follow this envelope structure:
```typescript
{
  v: number;              // Protocol version (currently 1)
  type: string;           // Message type (namespaced)
  id: string;             // Unique message ID
  sessionId: string;      // Conversation/session scope
  correlationId?: string; // Request/response pairing
  ts: string;             // ISO timestamp
  seq: number;            // Sequence number per session
  payload?: unknown;      // Type-specific payload
}
```

### Message Namespaces
- `agent:*` - AI agent lifecycle and streaming
- `fs:*` - File system operations
- `term:*` - Terminal I/O
- `ws:*` - WebSocket meta events
- `server:*` - Server status
- `discovery:*` - Server discovery

### Authentication Flow

1. **Pairing**:
   - Local (default):
     ```
     POST /auth/pair { deviceId, pin }
     → { deviceId, secret }
     ```
   - Remote (optional, requires one-time token):
     ```
     # Start with: pocket-server pair --remote
     POST /auth/pair { deviceId, pin, pairToken }
     → { deviceId, secret }
     ```

2. **Token Generation**:
   ```
   POST /auth/challenge { deviceId }
   → { nonce }
   
   POST /auth/token { deviceId, signature }
   → { token }
   ```
   
   Signature = `sha256(secret\ndeviceId\nnonce)`

3. **Usage**:
   - HTTP: `Authorization: Pocket <token>`
   - WebSocket: `ws://host/ws?token=<token>`

### Session Management

Sessions are stored in `data/sessions/<id>/`:
- `snapshot.json` - Current state
- `events.jsonl` - Event log (append-only)

Index at `data/sessions/index.json` for fast listing.

## Adding New Features

### New HTTP Endpoint
1. Create handler in appropriate module
2. Register route in module's index file:
   ```typescript
   router.POST('/path', async (req, res) => {
     // Handler logic
   });
   ```
3. Update `shared/types/api.ts` with request/response types
4. Add to mobile client's `ServerEndpoints` if needed

### New WebSocket Message Type
1. Define type in `shared/types/api.ts`
2. Add handler in `server/websocket.ts` or module handler
3. Emit via `wsManager.send()` or `wsManager.broadcast()`
4. Update mobile client's WebSocket handler

### New Tool for Agent
1. Create tool file in `agent/anthropic/tools/`
2. Implement tool interface with schema and handler
3. Register in `agent/anthropic/anthropic.ts`
4. Add approval logic for max mode if needed

## Performance Optimizations

### Terminal Streaming
- Frames aggregated every 8ms (120Hz target)
- Max frame size: 32KB for low latency
- 1MB backlog buffer per PTY session
- Automatic cleanup on disconnect

### WebSocket Management
- Heartbeat ping/pong every 30s
- Client-specific message routing
- Broadcast capability for system events
- Automatic reconnection handling

### Session Storage
- Write queue per session prevents conflicts
- Snapshot-based recovery for fast restarts
- Event log for audit trail
- Automatic index updates

## Security Considerations

### Path Validation
- All file operations restricted to HOME_DIR
- Path traversal prevention
- Symlink resolution checks

### Token Security
- Short-lived tokens (5 minutes)
- HMAC-SHA256 signatures
- Device-specific secrets
- Automatic token refresh

### Tool Safety
- Danger check for destructive operations
- Max mode allowlist for auto-approval
- Explicit user approval for sensitive tools
- Command timeout enforcement

## Testing Strategy

### Unit Tests
Location: Co-located with source files (`*.test.ts`)
```bash
npm test -- src/agent/anthropic/anthropic.test.ts
```

### Integration Points to Test
- WebSocket message flow
- Session persistence and recovery
- Tool execution and approval
- Auth token lifecycle
- Terminal I/O streaming

## Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=sk-ant-...    # Claude API key

# Optional
PORT=3000                        # Server port
CF_TUNNEL_TOKEN=...             # Cloudflare tunnel token
CF_TUNNEL_CONFIG=...            # Tunnel configuration
```

## CLI Operations

### Starting the Server
```bash
# Basic start
pocket-server start

# With remote access
pocket-server start --remote

# Custom port
pocket-server start --port 3010
```

### Pairing Devices
```bash
# Default 60s window
pocket-server pair

# Custom duration and PIN
pocket-server pair --duration=120000 --pin=9999

# Remote pairing (PIN + one-time token)
pocket-server pair --remote
```

### Maintenance
```bash
# Stop server
pocket-server stop

# Update to latest
pocket-server update

# Check status
pocket-server status
```

## Common Patterns

### Router Registration
```typescript
const router = createRouter('');
router.GET('/path', handler);
router.POST('/path', handler);
app.route('/prefix', router.getApp());
```

### WebSocket Broadcasting
```typescript
wsManager.broadcast({
  v: 1,
  type: 'event:type',
  id: crypto.randomUUID(),
  sessionId: 'system',
  ts: new Date().toISOString(),
  seq: 0,
  payload: data
});
```

### Session Store Usage
```typescript
const store = SessionStoreFs.getInstance();
await store.createSession(id, { workingDir, maxMode });
await store.recordUserMessage(id, content);
const snapshot = await store.getSnapshot(id);
```

### Tool Implementation
```typescript
export const toolName = {
  schema: { /* JSON schema */ },
  handler: async (params, context) => {
    // Implementation
    return { ok: true, result };
  }
};
```

## Debugging Tips

### Structured Logging
```typescript
logger.info('Module', 'action', { 
  sessionId, 
  messageId,
  details 
});
```

### WebSocket Inspection
- Monitor `ws:*` events in logs
- Check heartbeat responses
- Verify sequence numbers

### Session Debugging
- Check `data/sessions/<id>/snapshot.json`
- Review `events.jsonl` for history
- Verify index.json consistency

## Build and Release

### Local Build
```bash
npm run build
# Creates dist/index.js and dist/cli.js
```

### Release Process
1. Biome lint validation
2. Build with esbuild (ESM, Node 22)
3. Prune dev dependencies
4. Bundle Node runtime
5. Create tarball
6. Upload to Vercel Blob
7. Update manifest

### Distribution Layout
```
~/.pocket-server/
├── bin/                 # CLI symlink
├── current/            # Active version symlink
├── releases/           # Version directories
│   └── vX.Y.Z/
│       ├── node        # Bundled Node runtime
│       ├── dist/       # Server code
│       └── package.json
└── data/               # Persistent data
    ├── sessions/       # Conversation storage
    ├── auth/          # Device registry
    └── runtime/       # PID files
```

## Troubleshooting

### Common Issues

1. **Port already in use**
   - Check for existing process: `lsof -i :3000`
   - Use different port: `PORT=3001 npm run dev`

2. **WebSocket auth failures**
   - Verify token in query string
   - Check token expiry (5 minutes)
   - Ensure device is paired

3. **Terminal not responding**
   - Check PTY session exists
   - Verify frame aggregation working
   - Look for backlog overflow

4. **Session not found**
   - Verify session exists in `data/sessions/`
   - Check index.json for corruption
   - Ensure proper session creation

### Performance Monitoring
- Watch frame aggregation efficiency
- Monitor WebSocket message queue
- Check session write queue depth
- Track memory usage with large conversations

---
> Source: [yayasoumah/pocket-server](https://github.com/yayasoumah/pocket-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

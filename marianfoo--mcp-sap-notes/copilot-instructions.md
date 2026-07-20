## mcp-sap-notes

> An MCP (Model Context Protocol) server that gives AI assistants (Cursor, Claude, etc.) direct access to SAP Notes and Knowledge Base articles. It authenticates with SAP's systems via browser automation (Playwright) and exposes two tools: **search** and **get** for SAP Notes.

# SAP Note Search MCP Server - Project Overview for Claude Sessions

## What This Project Is

An MCP (Model Context Protocol) server that gives AI assistants (Cursor, Claude, etc.) direct access to SAP Notes and Knowledge Base articles. It authenticates with SAP's systems via browser automation (Playwright) and exposes two tools: **search** and **get** for SAP Notes.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  MCP Client (Cursor / Claude Desktop / LibreChat)           │
│  Communicates via JSON-RPC 2.0                              │
└───────────┬──────────────────────────────┬──────────────────┘
            │ stdio transport              │ HTTP transport
            ▼                              ▼
┌───────────────────────┐    ┌──────────────────────────────┐
│  src/mcp-server.ts    │    │  src/http-mcp-server.ts      │
│  StdioServerTransport │    │  Express + StreamableHTTP    │
│  (for Cursor / CLI)   │    │  (for Docker / LibreChat)    │
└───────────┬───────────┘    └──────────────┬───────────────┘
            │                               │
            └──────────┬────────────────────┘
                       ▼
         ┌──────────────────────────┐
         │  MCP Tool Handlers       │
         │  search         │
         │  fetch            │
         └────────────┬─────────────┘
                      │
        ┌─────────────┴──────────────┐
        ▼                            ▼
┌──────────────────┐    ┌────────────────────────┐
│  src/auth.ts     │    │  src/sap-notes-api.ts  │
│  SapAuthenticator│    │  SapNotesApiClient     │
│  Playwright +    │    │  Coveo Search API      │
│  Certificate/    │    │  Raw Notes API         │
│  Username auth   │    │  Fallback strategies   │
└──────────────────┘    └────────────────────────┘
        │                            │
        ▼                            ▼
┌──────────────────────────────────────────────┐
│  SAP Systems                                  │
│  • accounts.sap.com (IAS - authentication)    │
│  • me.sap.com (notes, search, Coveo token)    │
│  • Coveo search engine (search API)           │
│  • launchpad.support.sap.com (note URLs)      │
└──────────────────────────────────────────────┘
```

## Source Files

### Core Server Files

| File | Purpose | Key Classes/Exports |
|------|---------|-------------------|
| `src/mcp-server.ts` | **Main entry point** - stdio MCP server for Cursor/CLI | `SapNoteMcpServer` |
| `src/http-mcp-server.ts` | HTTP MCP server for Docker/LibreChat deployments | `HttpSapNoteMcpServer` |
| `src/auth.ts` | Authentication via Playwright browser automation | `SapAuthenticator` |
| `src/sap-notes-api.ts` | SAP Notes search (Coveo) and retrieval (raw API) | `SapNotesApiClient` |
| `src/types.ts` | TypeScript interfaces and JSON schemas | `ServerConfig`, `AuthState`, `SapNote`, etc. |
| `src/logger.ts` | Pino-based logging with MCP mode detection | `logger`, `authLogger`, `apiLogger` |
| `src/schemas/sap-notes.ts` | Enhanced Zod schemas with LLM-optimized descriptions | `NoteSearchInputSchema`, `NoteGetInputSchema`, etc. |

### Test Files

| File | Purpose |
|------|---------|
| `test/test-auth.js` | Tests authentication flow |
| `test/test-sap-api.js` | Tests Coveo search and note retrieval |
| `test/test-mcp-server.js` | Tests MCP protocol interaction |
| `test/test-docker-debug.js` | Docker environment debugging |

### Configuration

| File | Purpose |
|------|---------|
| `.env` / `.env.example` | Environment variables (PFX_PATH, PFX_PASSPHRASE, etc.) |
| `tsconfig.json` | TypeScript compilation (ES2022, ESNext modules, strict) |
| `token-cache.json` | Cached authentication cookies (auto-generated) |

## Authentication Flow

The server supports two authentication methods:

### 1. Certificate-Based (Original)
- Uses a `.pfx` client certificate for SAP IAS
- Playwright launches a browser, presents the certificate during TLS handshake
- SAP authenticates automatically, browser extracts session cookies
- Cookies cached to `token-cache.json` with configurable TTL

### 2. Username/Password (New)
- Uses SAP username + password for form-based login
- Playwright fills the login form on `accounts.sap.com`
- Supports MFA/2FA with configurable timeout for manual code entry
- Credentials passed via env vars (`SAP_USERNAME`, `SAP_PASSWORD`) or MCP client config
- Falls back to certificate auth if username/password not provided

### Auth Priority
1. Check `token-cache.json` for valid cached token
2. If `SAP_USERNAME` + `SAP_PASSWORD` provided → username/password auth
3. If `PFX_PATH` + `PFX_PASSPHRASE` provided → certificate auth
4. Error if neither configured

## MCP Tools

### `search`
- **Input:** `{ q: string, lang?: 'EN' | 'DE' }`
- **Flow:** Coveo search API → fallback to direct ID lookup → fallback to SAP internal search
- **Output:** Ranked list of matching SAP Notes with metadata

### `fetch`
- **Input:** `{ id: string, lang?: 'EN' | 'DE' }`
- **Flow:** Playwright raw notes API → fallback to HTTP raw notes API
- **Output:** Full note content (HTML), metadata, priority, category

## Key External APIs

| API | URL | Auth | Purpose |
|-----|-----|------|---------|
| Coveo Search | `sapamericaproductiontyfzmfz0.org.coveo.com/rest/search/v2` | Bearer token | Search SAP Notes |
| SAP Raw Notes | `me.sap.com/backend/raw/sapnotes/Detail` | Cookies | Fetch note content |
| SAP Search Page | `me.sap.com/search` | Cookies | Extract Coveo token from page |
| SAP IAS | `accounts.sap.com` | Certificate or form login | Authentication |

## Dependencies

| Package | Purpose |
|---------|---------|
| `@modelcontextprotocol/sdk` | Official MCP SDK (server, transports) |
| `playwright` | Browser automation for auth + note retrieval |
| `express` + `cors` | HTTP server mode |
| `zod` | Schema validation for MCP tools |
| `pino` + `pino-pretty` | Structured logging |
| `dotenv` | Environment variable loading |

## Running the Server

```bash
# Build
npm run build

# Stdio mode (for Cursor)
npm run serve

# HTTP mode (for Docker/LibreChat)
npm run serve:http

# Development with watch
npm run dev        # stdio
npm run dev:http   # http
```

## MCP Client Configuration

### Cursor / Claude Desktop (stdio)
```json
{
  "mcpServers": {
    "sap-notes": {
      "command": "node",
      "args": ["/path/to/mcp-sap-notes/dist/mcp-server.js"],
      "env": {
        "SAP_USERNAME": "your-sap-user",
        "SAP_PASSWORD": "your-sap-password"
      }
    }
  }
}
```

### HTTP Mode
```json
{
  "mcpServers": {
    "sap-notes": {
      "url": "http://localhost:3123/mcp",
      "headers": {
        "Authorization": "Bearer your-access-token"
      }
    }
  }
}
```

## Common Development Tasks

### Adding a new tool
1. Define Zod schemas in `src/schemas/sap-notes.ts`
2. Add tool description constant
3. Register tool in both `src/mcp-server.ts` and `src/http-mcp-server.ts` via `registerTool()`
4. Implement handler using `SapNotesApiClient` or `SapAuthenticator`

### Modifying authentication
- Auth logic is in `src/auth.ts` → `SapAuthenticator.authenticate()`
- Config loaded in server files' `loadConfig()` methods
- Token cache: `token-cache.json` at project root

### Updating tool descriptions
- Enhanced descriptions in `src/schemas/sap-notes.ts`
- These descriptions directly influence LLM tool selection accuracy
- Format uses USE WHEN / DO NOT USE WHEN / WORKFLOW PATTERN sections

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PFX_PATH` | For cert auth | - | Path to .pfx certificate |
| `PFX_PASSPHRASE` | For cert auth | - | Certificate passphrase |
| `SAP_USERNAME` | For user auth | - | SAP username (email) |
| `SAP_PASSWORD` | For user auth | - | SAP password |
| `MAX_JWT_AGE_H` | No | `12` | Token cache lifetime (hours) |
| `HEADFUL` | No | `false` | Show browser window |
| `LOG_LEVEL` | No | `info` | debug/info/warn/error |
| `HTTP_PORT` | No | `3123` | HTTP server port |
| `ACCESS_TOKEN` | No | - | Bearer token for HTTP mode |
| `MFA_TIMEOUT` | No | `120000` | MFA wait timeout (ms) |
| `AUTH_METHOD` | No | `auto` | Force auth method: `certificate`, `password`, `auto` |

---
> Source: [marianfoo/mcp-sap-notes](https://github.com/marianfoo/mcp-sap-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->

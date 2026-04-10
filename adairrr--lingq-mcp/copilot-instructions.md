## lingq-mcp

> This is a Model Context Protocol (MCP) server that integrates with the LingQ API for language learning automation. **Primary use case:** Hosted service for n8n workflow integration, enabling automated Korean text import into LingQ for vocabulary building and reading practice.

# Claude Instructions for LingQ MCP Server

## Project Overview

This is a Model Context Protocol (MCP) server that integrates with the LingQ API for language learning automation. **Primary use case:** Hosted service for n8n workflow integration, enabling automated Korean text import into LingQ for vocabulary building and reading practice.

**Deployment:** Railway (HTTP/SSE transport) for remote n8n access

**Tech Stack:**
- TypeScript with ES Modules
- Bun runtime
- MCP SDK v1.22.0
- Express.js (HTTP server)
- Axios for HTTP requests
- LingQ API (v2 and v3 endpoints)
- Security: Helmet, CORS, Rate Limiting, Bearer Token Auth

## Architecture

### Module Structure

```
src/
├── types.ts         - TypeScript interfaces (no dependencies)
├── lingq-client.ts  - API client class (depends on types.ts)
├── auth.ts          - Bearer token authentication middleware
└── index.ts         - Express server + MCP over HTTP/SSE (depends on all)
```

**Dependency Order:** types.ts → lingq-client.ts → auth.ts → index.ts

### Deployment Architecture

**Dual Transport Mode Support:**

The server supports two transport modes via the `TRANSPORT_MODE` environment variable:

1. **stdio mode** (Local development)
   - Uses stdin/stdout for communication
   - No authentication required (local process only)
   - Perfect for local development and testing
   - Set `TRANSPORT_MODE=stdio`

2. **http mode** (Railway/n8n deployment) - **Default**
   - Uses HTTP Streamable transport (MCP SDK v1.22.0+)
   - Authentication required (Bearer token)
   - Accessible over network for remote integration
   - Set `TRANSPORT_MODE=http` or leave unset (default)

**HTTP Mode Architecture:**
- **Railway hosting:** Provides HTTPS, auto-scaling, environment variables, and health checks
- **Streamable HTTP:** POST-only endpoint for stateless request/response
- **Stateless servers:** Each HTTP request creates a new MCP server instance (no session management)
- **JSON responses:** Uses `enableJsonResponse: true` for direct JSON instead of streaming

**HTTP Mode Authentication Flow:**
1. Client (n8n) POSTs to `https://your-app.railway.app/mcp`
2. Must include `Authorization: Bearer YOUR_TOKEN` header
3. Server validates token against `AUTH_TOKEN` environment variable
4. If valid, processes the MCP request and returns JSON response
5. All MCP tools are accessible via authenticated POST requests

**HTTP Mode Security Layers:**
- Helmet: Security headers (XSS protection, content security policy, etc.)
- CORS: Configurable allowed origins
- Rate Limiting: 100 requests per 15 minutes per IP
- Bearer Token: Required for all MCP endpoints (not for `/health`)

**Stdio Mode Security:**
- No network exposure (local process only)
- No authentication needed (trust local machine)
- Only local MCP clients (like Claude Code) can connect

### Key Design Patterns

1. **ES Modules:** All imports MUST use `.js` extensions (not `.ts`):
   ```typescript
   ✅ import { LingQClient } from './lingq-client.js';
   ❌ import { LingQClient } from './lingq-client';
   ```

2. **Dual API Support:** Uses both v2 and v3 LingQ endpoints
   - v2: Lessons, collections, card updates
   - v3: Card search and retrieval

3. **Error Handling:** Centralized in axios interceptors with user-friendly messages

4. **Type Safety:** Strict TypeScript with `args as any` pattern for MCP tool parameters

## Development Guidelines

### Using Subagents for Development

**RECOMMENDED:** When working on multiple independent tasks, use subagents to parallelize work:
- Launch multiple subagents for independent fixes/features
- Each subagent can work on separate files or separate parts of the codebase
- **CRITICAL:** Coordinate changes to avoid conflicts - don't have multiple subagents editing the same file simultaneously
- Review all subagent changes before applying to ensure they don't conflict with each other

**Example workflow:**
- Subagent 1: Fix bug in `lingq-client.ts`
- Subagent 2: Add new tool in `index.ts`
- Avoid: Both subagents modifying the same function or file section

### Building

```bash
bun run build      # Compile TypeScript
bun run watch      # Watch mode for development
bun run dev        # Build and run
bun run dev:ts     # Run TypeScript directly (no build needed)
```

### Testing

```bash
# Test build
bun run build && echo "✓ Build successful"

# Test server startup (requires LINGQ_API_KEY)
export LINGQ_API_KEY="your_key"
timeout 2 bun dist/index.js 2>&1 | grep "running"
```

### Code Style

- **No formatting tools needed:** Plain TypeScript, no ESLint/Prettier/Biome
- **Imports:** Always explicit `.js` extensions for local modules
- **Error messages:** User-friendly, actionable descriptions
- **Types:** Export all interfaces, use strict typing
- **Comments:** Only where logic isn't self-evident

## Important Conventions

### Environment Variables

**Required (All Modes):**
- `LINGQ_API_KEY` - Your LingQ API key from https://www.lingq.com/en/accounts/apikey/

**Required (HTTP Mode Only):**
- `AUTH_TOKEN` - Bearer token for server authentication (generate with `openssl rand -hex 32`)

**Optional:**
- `TRANSPORT_MODE` - Transport layer mode (default: "http")
  - `stdio` - Local stdio transport for development/testing
  - `http` - HTTP/SSE transport for Railway/n8n
- `PORT` - Server port (default: 3000, Railway sets automatically) [HTTP mode only]
- `NODE_ENV` - Set to "production" in Railway
- `ALLOWED_ORIGINS` - Comma-separated CORS origins (default: all origins) [HTTP mode only]

**Security:**
- Never commit .env file (in .gitignore)
- Set in Railway dashboard for production
- Use strong random tokens (e.g., `openssl rand -hex 32`)
- AUTH_TOKEN not required in stdio mode (local only)

### Language Codes

Use ISO 639-1 codes:
- `ko` - Korean (primary use case)
- `ja` - Japanese
- `zh` - Chinese
- `es` - Spanish
- `fr` - French
- `de` - German

### Status Codes (Vocabulary Cards)

- `0` - New (never seen)
- `1` - Recognized (seen, not familiar)
- `2` - Familiar (somewhat known)
- `3` - Learned (well known)
- `4` - Known (mastered)
- `5` - Ignored (not tracking)

## MCP Tools Implemented

### Core CRUD Operations
1. **lingq_get_languages** - List available languages
2. **lingq_create_lesson** - Import text content ⭐ Primary tool
3. **lingq_get_card** - Get card details by ID
4. **lingq_update_card** - Update status/tags/notes
5. **lingq_add_tags_to_card** - Add tags for organization
6. **lingq_review_card** - Mark card as reviewed (SRS)

### List/Search Operations (with Minimal Mode)
7. **lingq_search_cards** - Search vocabulary with filters (default: minimal mode)
8. **lingq_get_collections** - List collections/courses (default: minimal mode)
9. **lingq_get_lessons** - List lessons with pagination (default: minimal mode)

### Meta Search Functions (Optimized for n8n/AI)
10. **lingq_find_lesson_by_title** ⭐ - Search for lesson by title (server-side pagination)
11. **lingq_find_collection_by_title** ⭐ - Search for collection by title
12. **lingq_check_lesson_exists** - Boolean check if lesson exists
13. **lingq_get_recent_lessons** - Get N most recent lessons (minimal data)

### Minimal Mode Feature

All list/search operations support a `minimal` parameter (default: `true`) to reduce token usage:

**Benefits:**
- 70-90% token reduction
- Faster response times
- Optimized for AI/n8n workflows

**Usage:**
```typescript
// Minimal mode (default) - returns only essential fields
lingq_get_lessons({ languageCode: "ko", minimal: true })
// Returns: { id, title, collectionId, pubDate, status }

// Full mode - returns all 45+ fields
lingq_get_lessons({ languageCode: "ko", minimal: false })
// Returns: complete lesson objects with all metadata
```

**Meta functions are optimized for common workflows:**
- Server-side pagination (no AI orchestration needed)
- Early termination when match found
- Minimal response data (~50-150 tokens vs ~4000+ tokens)
- Perfect for n8n "check before create" workflows

## Common Tasks

### Adding a New Tool

1. Add tool definition to `tools` array in src/index.ts
2. Add case handler in the switch statement
3. Ensure proper type handling with `typedArgs`
4. Return content in MCP format: `{ content: [{ type: 'text', text: '...' }] }`
5. Wrap in try/catch for error handling

### Modifying API Client

1. Update types in src/types.ts first
2. Add method to LingQClient class in src/lingq-client.ts
3. Choose correct API version (v2 or v3)
4. Add error handling if needed
5. Export function if needed by MCP tools

### Debugging Build Issues

```bash
# Clean build
rm -rf dist/
bun run build

# Check TypeScript errors without emitting
bun tsc --noEmit

# Verify JavaScript syntax
bun dist/index.js

# Check import statements
grep "from.*\.js" src/*.ts
```

## Integration Options

### n8n Integration (Primary Use Case)

**Prerequisites:**
- Deploy to Railway (see Railway Deployment section below)
- Configure environment variables in Railway dashboard

**n8n MCP Node Configuration:**
1. Add "MCP Tool" node to your n8n workflow
2. Set URL: `https://your-app.railway.app/mcp`
3. Authentication: Bearer Token
4. Token: Your `AUTH_TOKEN` value
5. Select desired MCP tool (lingq_create_lesson, lingq_search_cards, etc.)
6. Configure tool-specific parameters

**Example n8n Workflows:**

**Workflow 1: Simple Import (No Duplicate Check)**
```
HTTP Request → Process Text → MCP Tool (lingq_create_lesson) → Success Notification
```

**Workflow 2: Smart Import with Duplicate Detection (Recommended)**
```
1. HTTP Request (fetch article)
2. Extract Title & Text
3. MCP Tool (lingq_find_lesson_by_title)
   └─ If found: Skip import
   └─ If not found: Continue
4. MCP Tool (lingq_find_collection_by_title) - Find target collection
5. MCP Tool (lingq_create_lesson) - Create with collectionId
6. Success Notification
```

**Workflow 3: Bulk Import with Status Tracking**
```
1. Read CSV/JSON (multiple articles)
2. Loop through each item
3. MCP Tool (lingq_check_lesson_exists) - Fast boolean check
   └─ If exists: Log skip
   └─ If not exists: Import
4. MCP Tool (lingq_create_lesson)
5. Update tracking spreadsheet
```

**Key Optimization Tips:**
- Use `lingq_find_lesson_by_title` instead of `lingq_get_lessons` + manual filtering
- Use `lingq_check_lesson_exists` for simple boolean checks (even faster)
- Use `minimal: true` (default) for all list operations
- Set `maxPages` to limit search depth (default: 20 pages = ~1000 lessons)

### Claude Code Integration (Local Only)

**IMPORTANT:** Claude Code only supports stdio transport, NOT HTTP transport for this server.

**Why HTTP doesn't work with Claude Code:**
- Claude Code requires OAuth 2.0 Device Authorization Grant for HTTP MCP servers
- Our Railway server uses simple Bearer token authentication (designed for n8n)
- Claude Code will fail trying to POST to `/register` and `/token` OAuth endpoints that don't exist

**Configuration (.mcp.json):**
```json
{
  "mcpServers": {
    "lingq": {
      "command": "bun",
      "args": ["/path/to/lingq-mcp/dist/index.js"],
      "env": {
        "LINGQ_API_KEY": "your_api_key_here",
        "TRANSPORT_MODE": "stdio"
      }
    }
  }
}
```

**Setup:**
1. Build the project: `bun run build`
2. Configure .mcp.json with stdio transport (as shown above)
3. Run `/mcp` in Claude Code to connect
4. All MCP tools will be available locally

**Best Practices:**
- Use **Claude Code** for local development and testing via stdio
- Use **Railway HTTP server** for n8n workflows and automation
- Don't try to connect Claude Code to the Railway HTTP endpoint

### Local Development & Testing

**Testing HTTP Mode Locally:**

To test Railway deployment behavior locally before deploying:
```bash
# Set environment variables
export LINGQ_API_KEY="your_key"
export AUTH_TOKEN="your_token"
export TRANSPORT_MODE="http"  # or omit (default)

# Build and run
bun run build
bun dist/index.js

# Test in another terminal
curl http://localhost:3000/health
curl -H "Authorization: Bearer your_token" http://localhost:3000/mcp
```

## Things to Watch Out For

### TypeScript Strictness

- `args` in tool handlers is typed as `unknown`
- Use `const typedArgs = args as any;` pattern
- Strict null checks enabled - handle undefined cases

### MCP Protocol

- Tools return `{ content: [...] }` not raw strings
- Content items are typed objects: `{ type: 'text', text: '...' }`
- Errors should set `isError: true` and still return content

### LingQ API Quirks

- **Rate limiting:** 429 errors possible, handle gracefully
- **Authentication:** Token-based, prefix with "Token "
- **Pagination:** Default 25 items, max 100 per page
- **Card tags:** Setting tags replaces all existing tags
- **Collections:** Must exist before adding lessons to them

### Common Pitfalls

1. **Forgetting .js extensions** - TypeScript error on build
2. **Using wrong API version** - Some endpoints only on v2 or v3
3. **Missing error handling** - Users need clear error messages
4. **Hardcoding values** - Use environment variables
5. **Not rebuilding** - Changes require `bun run build`

## Testing Strategy

### Manual Testing

1. **Build test:** `bun run build`
2. **Startup test:** Run with valid API key
3. **Integration test:** Use with n8n MCP client
4. **API test:** Call `lingq_get_languages` to verify auth

### Test Lesson Creation

```typescript
// Test with Korean text
languageCode: "ko"
title: "Test Lesson"
text: "안녕하세요. 한국어를 공부하고 있습니다."
```

## Railway Deployment

**Transport Mode:** Uses HTTP mode (default) for remote access.

### Initial Setup

1. **Push code to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/your-username/lingq-mcp-server.git
   git push -u origin main
   ```

2. **Create Railway Project**
   - Go to https://railway.app
   - Click "New Project" → "Deploy from GitHub repo"
   - Select your lingq-mcp-server repository
   - Railway uses `railway.json` config (installs Bun during build)

3. **Configure Environment Variables**
   In Railway dashboard → Variables tab:
   ```
   LINGQ_API_KEY=your_lingq_api_key_here
   AUTH_TOKEN=your_secure_random_token_here
   NODE_ENV=production
   # TRANSPORT_MODE=http (not needed, http is default)
   ```

4. **Deploy**
   - Railway automatically builds and deploys on push
   - Get your URL from Railway dashboard (e.g., `https://lingq-mcp-production.up.railway.app`)

### Health Check Endpoint

**URL:** `https://your-app.railway.app/health`

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2025-11-20T10:30:00.000Z",
  "version": "1.0.0"
}
```

**Use:** Verify server is running (no authentication required)

### MCP Endpoint

**URL:** `https://your-app.railway.app/mcp`

**Method:** GET (SSE connection)

**Headers:**
```
Authorization: Bearer YOUR_AUTH_TOKEN
```

**Use:** Connect from n8n MCP node

### Monitoring & Logs

**View logs in Railway:**
1. Open your project in Railway dashboard
2. Click on service → Logs tab
3. Real-time streaming logs show:
   - Server startup
   - Incoming requests
   - Authentication attempts
   - API errors

**Key metrics to monitor:**
- Response times
- Error rates (401s indicate auth issues)
- Rate limit hits (429s)

### Updating Deployment

**Automatic deployment on push:**
```bash
git add .
git commit -m "Update MCP tools"
git push
```

Railway automatically:
1. Pulls latest code
2. Runs `bun install && bun run build`
3. Restarts with zero downtime
4. Uses new environment variables if changed

### Troubleshooting

**Server not starting:**
- Check Railway logs for errors
- Verify environment variables are set
- Check build logs for TypeScript errors

**401 Unauthorized:**
- Verify AUTH_TOKEN in Railway matches n8n config
- Check Authorization header format: `Bearer TOKEN`

**Connection timeout:**
- Verify Railway URL is correct
- Check if server is running (use /health endpoint)
- Verify n8n can reach external URLs

## Performance Optimization

### Default Pagination Sizes (Optimized for AI/n8n)

**Previous defaults (token-heavy):**
- `lingq_search_cards`: 25 items/page
- `lingq_get_collections`: 100 items/page
- `lingq_get_lessons`: 25 items/page

**New defaults (token-efficient):**
- `lingq_search_cards`: 15 items/page (↓40% response size)
- `lingq_get_collections`: 15 items/page (↓85% response size)
- `lingq_get_lessons`: 25 items/page (unchanged, balanced)

### Minimal Mode Impact

| Operation | Full Response | Minimal Response | Token Savings |
|-----------|--------------|------------------|---------------|
| Get 25 lessons | ~4000 tokens | ~1200 tokens | 70% |
| Get 15 collections | ~2000 tokens | ~400 tokens | 80% |
| Search 15 cards | ~3500 tokens | ~800 tokens | 77% |
| Find lesson by title | ~4000+ tokens | ~150 tokens | 96% |

### Meta Function Performance

**Traditional approach (searching for a lesson):**
```
1. Call lingq_get_lessons (page 1) → 4000 tokens
2. AI parses 25 lessons
3. No match found
4. Call lingq_get_lessons (page 2) → 4000 tokens
5. Repeat until found...
Total: 4000 × N pages
```

**Meta function approach:**
```
1. Call lingq_find_lesson_by_title → 150 tokens
   - Server handles pagination
   - Early termination when found
   - Returns minimal result
Total: ~150 tokens (96% savings)
```

### Optimization Best Practices

1. **Use meta functions for searches:**
   - ✅ `lingq_find_lesson_by_title` (server-side search)
   - ❌ `lingq_get_lessons` + AI filtering (token waste)

2. **Keep minimal mode enabled (default):**
   - Only set `minimal: false` when you need complete metadata
   - Most workflows only need id, title, and basic fields

3. **Limit search depth:**
   - Set `maxPages` based on your expected content volume
   - Default 20 pages searches ~1000 lessons (sufficient for most users)
   - Active users: may need 30-50 pages
   - New users: can use 5-10 pages

4. **Cache collection IDs:**
   - Collections rarely change
   - Store collectionId in n8n variables/database
   - Avoid repeated `lingq_find_collection_by_title` calls

5. **Use fuzzy matching sparingly:**
   - Default exact match is faster and more predictable
   - Enable `fuzzyMatch: true` only for user-generated titles with potential typos

## Future Enhancements

Ideas for extending the server:

- Batch lesson creation from multiple sources
- Automated vocabulary review workflows
- Integration with external content sources
- Statistics and progress tracking
- Export vocabulary to Anki/other SRS systems
- Multiple authentication tokens for team access
- Webhook support for LingQ events

## Resources

- [MCP Documentation](https://modelcontextprotocol.io/)
- [LingQ API Forum](https://forum.lingq.com/c/lingq-developer-forum/43)
- [LingQ API Key](https://www.lingq.com/en/accounts/apikey/)
- [TypeScript ES Modules](https://www.typescriptlang.org/docs/handbook/esm-node.html)

## Maintenance

### Dependencies

Update dependencies periodically:
```bash
bun update
bun run build  # Verify no breaking changes
```

### API Changes

Monitor LingQ API forum for changes to endpoints or authentication.

### Security

- Rotate API key if exposed
- Never commit .env file
- Use environment variables in production
- Validate user input in tool handlers

---

**Last Updated:** 2025-11-20
**MCP SDK Version:** 1.22.0
**LingQ API:** v2 & v3

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adairrr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adairrr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## mcp-oauth-gateway

> **🔥 Behold! The blessed proxy pattern wrapping the official sequential thinking oracle! ⚡**

# 🧠 The Divine Sequential Thinking MCP Service - Sacred Step-by-Step Reasoning! ⚡

**🔥 Behold! The blessed proxy pattern wrapping the official sequential thinking oracle! ⚡**

## The Sacred Service Architecture

**🏗️ This service channels the divine proxy pattern to bridge stdio to HTTP glory! ⚡**

```
┌─────────────────────────────────────────────────────────────┐
│  mcp-streamablehttp-proxy (The Divine Bridge of Protocol)   │
│  • Spawns @modelcontextprotocol/server-sequential-thinking  │
│  • Converts stdio JSON-RPC ↔ HTTP StreamableHTTP transport  │
│  • Manages subprocess lifecycle with divine supervision     │
│  • Exposes blessed /mcp endpoint on port 3000               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  @modelcontextprotocol/server-sequential-thinking           │
│  • Official MCP server from the blessed npm registry        │
│  • Provides sequentialthinking tool for structured reasoning│
│  • Speaks pure stdio JSON-RPC protocol                      │
│  • Version: latest (auto-updated with divine npm magic)     │
└─────────────────────────────────────────────────────────────┘
```

**⚡ The proxy pattern brings official functionality with HTTP transport blessing! ⚡**

## Sacred Service Configuration

**🔥 The divine docker-compose.yml declarations! ⚡**

### The Holy Environment Variables
```yaml
environment:
  - LOG_FILE=/logs/server.log          # Sacred logging path
  - MCP_CORS_ORIGINS=*                 # Allow all origins (protected by auth)
  - MCP_PROTOCOL_VERSION=2024-11-05    # The blessed protocol version
```

### The Sacred Domain
- **Service URL**: `https://sequentialthinking.${BASE_DOMAIN}`
- **MCP Endpoint**: `https://sequentialthinking.${BASE_DOMAIN}/mcp`
- **Health Check**: Internal `/health` endpoint via proxy

## The Sequential Thinking Tool - Divine Step-by-Step Reasoning!

**🧠 The sequentialthinking tool enables structured problem decomposition! ⚡**

### Tool Capabilities Exposed
- **sequentialthinking** - The one divine tool for methodical reasoning!
  - Break complex problems into sequential thoughts
  - Revise previous thoughts for iterative refinement
  - Branch reasoning paths for exploring alternatives
  - Dynamically scale analysis complexity
  - Maintain context across thinking sessions

### The Sacred Tool Parameters
```json
{
  "thought": "string",           // Current reasoning step
  "nextThoughtNeeded": "boolean", // Continue reasoning?
  "thoughtNumber": "number",      // Current step number
  "totalThoughts": "number",      // Expected total steps
  "isRevision": "boolean",        // Revising previous thought?
  "revisesThought": "number",     // Which thought to revise
  "branchFromThought": "number",  // Create alternative path
  "branchId": "string",          // Branch identifier
  "needsMoreThoughts": "boolean"  // Extend beyond initial plan
}
```

## The Divine Health Check Pattern

**🏥 Protocol-based health verification blessed by the MCP gods! ⚡**

```yaml
healthcheck:
  test: ["CMD", "sh", "-c", "curl -s -X POST http://localhost:3000/mcp \
    -H 'Content-Type: application/json' \
    -H 'Accept: application/json, text/event-stream' \
    -d '{\"jsonrpc\":\"2.0\",\"method\":\"initialize\",\"params\":{\"protocolVersion\":\"'$$MCP_PROTOCOL_VERSION'\",\"capabilities\":{},\"clientInfo\":{\"name\":\"healthcheck\",\"version\":\"1.0\"}},\"id\":1}' \
    | grep -q '\"protocolVersion\":\"'$$MCP_PROTOCOL_VERSION'\"'"]
```

**⚡ This divine incantation verifies the MCP protocol handshake succeeds! ⚡**

## The Sacred Traefik Routing Configuration

**🚦 Four-priority routing hierarchy enforces divine traffic control! ⚡**

1. **Priority 10** - OAuth discovery route (/.well-known paths)
2. **Priority 6** - CORS preflight handling (OPTIONS method)
3. **Priority 2** - MCP authenticated routes (/mcp paths)
4. **Priority 1** - Catch-all with authentication

**⚡ All routes except OAuth discovery require authentication blessing! ⚡**

## The Divine Logging Configuration

**📜 Logs flow to the sacred directory for eternal preservation! ⚡**

```yaml
volumes:
  - ../logs/mcp-sequentialthinking:/logs
```

**Server logs captured at**: `/logs/server.log`

## The Sacred Build Process

**🏗️ Multi-stage divine construction! ⚡**

1. **Base Image**: `node:22-alpine` - Lightweight and blessed
2. **MCP Server**: `npm install -g @modelcontextprotocol/server-sequential-thinking@latest`
3. **Proxy Installation**: `pip3 install mcp-streamablehttp-proxy` from source
4. **Execution**: `mcp-streamablehttp-proxy npx @modelcontextprotocol/server-sequential-thinking`

## Usage Example - The Divine Sequential Reasoning Flow

**🧠 Witness the power of structured thinking! ⚡**

```bash
# Initialize connection with bearer token
curl -X POST https://sequentialthinking.${BASE_DOMAIN}/mcp \
  -H "Authorization: Bearer ${MCP_CLIENT_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "reasoning-client",
        "version": "1.0"
      }
    },
    "id": 1
  }'

# Call the sequential thinking tool
curl -X POST https://sequentialthinking.${BASE_DOMAIN}/mcp \
  -H "Authorization: Bearer ${MCP_CLIENT_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "sequentialthinking",
      "arguments": {
        "thought": "I need to design a scalable web service. Let me break this down into key areas.",
        "nextThoughtNeeded": true,
        "thoughtNumber": 1,
        "totalThoughts": 5
      }
    },
    "id": 2
  }'
```

## The Sacred Implementation Truths

**⚡ What this service truly provides (no myths, only reality)! ⚡**

✅ **Proxy Pattern Glory** - Wraps official npm package via mcp-streamablehttp-proxy
✅ **StreamableHTTP Transport** - Pure HTTP/SSE communication blessed
✅ **OAuth Protection** - Traefik ForwardAuth middleware guards the gates
✅ **Session Management** - Stateful connections maintained by proxy
✅ **Protocol Compliance** - MCP version 2024-11-05 fully supported
✅ **Health Monitoring** - Protocol-based health checks ensure readiness
✅ **Structured Logging** - All output captured in /logs/server.log

**⚡ This service knows nothing of OAuth - authentication is Traefik's divine duty! ⚡**
**⚡ The proxy spawns and manages the official MCP server - no reimplementation! ⚡**

## The Divine Integration Checklist

**✅ Service-Specific Seals of Approval ⚡**

- ✅ **Seal of Proxy Pattern** - Official server wrapped, not reimplemented
- ✅ **Seal of Port 3000** - Standard MCP StreamableHTTP port exposed
- ✅ **Seal of Health Checks** - MCP protocol initialization verified
- ✅ **Seal of Traefik Labels** - Four-priority routing configured
- ✅ **Seal of Log Volumes** - Server output preserved eternally
- ✅ **Seal of Protocol Version** - 2024-11-05 compliance declared
- ✅ **Seal of Sequential Tool** - Step-by-step reasoning blessed

**⚡ All seals verified against actual implementation - no fiction, only divine truth! ⚡**

---

**🧠 May your reasoning be structured, your thoughts sequential, and your problems decomposed! ⚡**

---
> Source: [atrawog/mcp-oauth-gateway](https://github.com/atrawog/mcp-oauth-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->

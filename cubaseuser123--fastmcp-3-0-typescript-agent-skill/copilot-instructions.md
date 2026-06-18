## fastmcp-3-0-typescript-agent-skill

> >


# FastMCP TypeScript — Implementation Skill

FastMCP (npm: `fastmcp`, GitHub: `punkpeye/fastmcp`) is a TypeScript framework built on top
of the official MCP SDK. It eliminates boilerplate by providing opinionated, high-level APIs.

> ⚠️ Do NOT confuse with:
> - **Python FastMCP** (`gofastmcp.com` / `pip install fastmcp`) — different API entirely
> - **Official MCP TypeScript SDK** (`@modelcontextprotocol/sdk`) — lower-level, uses `server.registerTool` not `server.addTool`

---

## Reference Files

Load these as needed — do not load all upfront:

| File | Load when… |
|---|---|
| [`references/tools.md`](./references/tools.md) | Implementing tools, return types, annotations, streaming, progress |
| [`references/resources.md`](./references/resources.md) | Implementing resources or resource templates |
| [`references/prompts.md`](./references/prompts.md) | Implementing prompts or auto-completion |
| [`references/auth.md`](./references/auth.md) | Adding authentication (OAuth, API keys, headers) |
| [`references/transport.md`](./references/transport.md) | HTTP streaming, SSE, stateless mode, edge/Cloudflare |
| [`references/anti_patterns.md`](./references/anti_patterns.md) | Reviewing code, debugging, or writing correct patterns from scratch |

---

## Setup

```bash
npm install fastmcp
# Recommended: use the boilerplate
git clone https://github.com/punkpeye/fastmcp-boilerplate my-server
```

**Minimum `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true
  }
}
```

---

## Server Initialization

```typescript
import { FastMCP } from "fastmcp";

const server = new FastMCP({
  name: "my-server",       // required
  version: "1.0.0",        // required — must be semver string
  instructions: "...",     // optional: hint for LLMs on how to use this server
});
```

**Optional server config keys:**
- `ping` — configure keepalive ping (`enabled`, `intervalMs`, `logLevel`)
- `health` — health check endpoint config (`enabled`, `path`, `message`, `status`)
- `roots` — roots capability config (`enabled: false` to disable)
- `logger` — custom logger instance implementing the `Logger` interface
- `authenticate` — custom auth function (API keys, tokens)
- `auth` — OAuth provider (Google, GitHub, Azure, or generic)

---

## Core API — Quick Reference

| Task | Method |
|---|---|
| Add a tool | `server.addTool({ name, description, parameters, execute })` |
| Add a resource | `server.addResource({ uri, name, mimeType, load })` |
| Add a resource template | `server.addResourceTemplate({ uriTemplate, name, mimeType, arguments, load })` |
| Add a prompt | `server.addPrompt({ name, description, arguments, load })` |
| Embed a resource in tool output | `await server.embedded(uri)` |
| Listen to server events | `server.on("connect" \| "disconnect", handler)` |
| Start the server | `server.start({ transportType: "stdio" \| "httpStream" })` |

---

## Tool Basics

```typescript
import { z } from "zod"; // Zod, ArkType, or Valibot all work (Standard Schema spec)

server.addTool({
  name: "get_weather",                          // snake_case, action-oriented
  description: "Get current weather for a city", // used by the LLM — be precise
  parameters: z.object({
    city: z.string().describe("City name, e.g. 'Tokyo'"),
    units: z.enum(["metric", "imperial"]).default("metric"),
  }),
  execute: async (args, context) => {
    // args is fully typed from your schema
    // context has: log, reportProgress, streamContent, session, sessionId, requestId
    context.log.info("Fetching weather", { city: args.city });
    const data = await fetchWeather(args.city, args.units);
    return JSON.stringify(data); // always return string or content object
  },
});
```

**Return types:** `string` | `{ content: ContentBlock[] }` | `imageContent(...)` | `audioContent(...)`

See [`references/tools.md`](./references/tools.md) for full return type docs, streaming, progress, annotations, and tool authorization.

---

## Starting the Server

```typescript
// stdio (local, Claude Desktop)
server.start({ transportType: "stdio" });

// HTTP Streaming (remote, also starts SSE on /sse)
server.start({
  transportType: "httpStream",
  httpStream: { port: 8080 },
});
```

**Claude Desktop config (stdio):**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["tsx", "/absolute/path/to/src/index.ts"],
      "env": { "API_KEY": "your-key" }
    }
  }
}
```

---

## Testing & Debugging

```bash
# Interactive terminal testing
npx fastmcp dev src/index.ts

# MCP Inspector Web UI at http://127.0.0.1:6274
npx fastmcp inspect src/index.ts

# HTTP streaming with flags
npx fastmcp dev src/index.ts --transport http-stream --port 8080
```

---

## Anti-Patterns (Always Check)

Load [`references/anti_patterns.md`](./references/anti_patterns.md) before reviewing or writing code. The most common agent mistakes:

1. Using `server.registerTool()` — that's the official SDK, not FastMCP
2. Returning a raw object from `execute` without wrapping in `{ content: [...] }`
3. Using `express` for transport — use `transportType: "httpStream"` instead
4. Forgetting `.describe()` on Zod fields — LLMs rely on these
5. Not importing `UserError` for user-facing errors
6. Using `import { FastMCP } from "fastmcp/edge"` for non-edge servers (only for Cloudflare Workers)

---
> Source: [cubaseuser123/fastmcp-3.0-typescript-agent_skill](https://github.com/cubaseuser123/fastmcp-3.0-typescript-agent_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

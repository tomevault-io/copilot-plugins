## mcp-remote

> For remote servers, start a web server with a Server-Sent Events (SSE) endpoint, and a separate endpoint for the client to send its messages to:

### HTTP with SSE

For remote servers, start a web server with a Server-Sent Events (SSE) endpoint, and a separate endpoint for the client to send its messages to:

```typescript
import express from "express";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
});

// ... set up server resources, tools, and prompts ...

const app = express();

app.get("/sse", async (req, res) => {
  const transport = new SSEServerTransport("/messages", res);
  await server.connect(transport);
});

app.post("/messages", async (req, res) => {
  // Note: to support multiple simultaneous connections, these messages will
  // need to be routed to a specific matching transport. (This logic isn't
  // implemented here, for simplicity.)
  await transport.handlePostMessage(req, res);
});

app.listen(3001);
```

---
> Source: [chriscarrollsmith/taskqueue-mcp](https://github.com/chriscarrollsmith/taskqueue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->

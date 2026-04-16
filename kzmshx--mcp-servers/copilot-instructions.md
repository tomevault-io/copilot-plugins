## mcp-servers

> MCP （Model Context Protocol）サーバーの実装パターン

# Implement the MCP server as follows

```ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// Define server
const server = new McpServer({ name: "Some MCP Server", version: "0.1.0" });

// Define tool
server.tool(
  "<name>",
  "<description>",
  // Input schema
  {
    foo: z.string(),
    bar: z.string().optional(),
  },
  // Implementation
  async ({ foo, bar }) => {
    // Run something...
    
    // If failure
    if (err) {
      return {
        content: [{ type: "text", text: "<error_response>" }],
        isError: true,
      }
    }

    // Then return response
    return {
      content: [{ type: "text", text: "<success_response>" }],
    };
  }
);

// Define prompt
server.prompt(
  "review-code",
  {
    code: z.string()
  },
  ({ code }) => ({
    messages: [{
      role: "user",
      content: {
        type: "text",
        text: `Please review this code:\n\n${code}`
      }
    }]
  })
);

// Define transport
const transport = new StdioServerTransport();
await server.connect(transport);
```

# If environments are needed, parse them into a top-level variable using Zod schema

```ts
import { z } from "zod";

// Define schema
const envSchema = z.object({
  HOGE: z.string().min(1, "HOGE is required"),
})

// Parse and assign env vars
const env = envSchema.parse(process.env);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kzmshx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

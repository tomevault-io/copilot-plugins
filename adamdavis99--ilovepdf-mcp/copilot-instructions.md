## ilovepdf-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that exposes iLovePDF API operations as tools for LLM clients. The server bridges Claude and other MCP-compatible clients to PDF processing capabilities.

## Development Commands

```bash
# Install dependencies
npm install

# TypeScript compilation (once tsconfig.json is configured)
npx tsc

# Run the server (after building)
node dist/index.js

# Run with tsx during development
npx tsx src/index.ts
```

## Architecture

### Core Dependencies

- **@modelcontextprotocol/sdk**: MCP server implementation providing tools, resources, and transports
- **@ilovepdf/ilovepdf-nodejs**: Node.js client for iLovePDF API

### iLovePDF Task Pattern

All PDF operations follow the same async workflow:

```typescript
const instance = new ILovePDFApi(publicKey, secretKey);
const task = instance.newTask('merge'); // or compress, split, etc.

await task.start();
await task.addFile(urlOrPath);  // Can add multiple files
await task.process();
const data = await task.download();
```

### Available PDF Operations

`merge`, `split`, `compress`, `rotate`, `repair`, `unlock`, `protect`, `officepdf`, `imagepdf`, `htmlpdf`, `pdfjpg`, `pdfa`, `watermark`, `pagenumber`, `editpdf`, `extract`, `pdfocr`, `validatepdfa`, `sign`

### MCP Server Pattern

Use `McpServer` from the SDK with tool handlers:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({ name: "ilovepdf", version: "1.0.0" });

server.tool("merge-pdfs", { /* zod schema */ }, async (params) => {
  // Implementation
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Transport Options

- **stdio**: For local Claude Desktop or CLI integrations (recommended for this use case)
- **Streamable HTTP**: For remote server deployments
- **SSE**: Legacy/backwards compatibility only

## Environment Variables

The server will need iLovePDF API credentials:

- `ILOVEPDF_PUBLIC_KEY`: Public key from developer.ilovepdf.com
- `ILOVEPDF_SECRET_KEY`: Secret key from developer.ilovepdf.com

## Key Implementation Notes

- iLovePDF operations are asynchronous and involve file uploads to their servers
- Each task has `remainingFiles` property after start() to check API quota
- Files can be added via URL or local path using `ILovePDFFile` class
- PDF info (pageNumber, pageSizes) available after addFile() resolves
- Zod is a required peer dependency for MCP SDK schema validation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamdavis99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adamdavis99)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

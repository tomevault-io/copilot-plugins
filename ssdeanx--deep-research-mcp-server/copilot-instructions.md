## 03-mcp-server

> MCP Server Rules (src/mcp-server.ts) - These rules align `src/mcp-server.ts` with @modelcontextprotocol/sdk best practices and your project’s research pipeline.


# MCP Server Rules (src/mcp-server.ts)

Last reviewed: 2025-08-11
Timestamp: 2025-08-11T08:21:00-04:00

These rules align `src/mcp-server.ts` with @modelcontextprotocol/sdk best practices and your project’s research pipeline.

## Scope & Goals
- Ensure the server exposes a clean, testable tool interface for deep research.
- Favor stdio transport for local/CLI and consider Streamable HTTP for deployment.
- Keep outputs deterministic, cache-aware, and friendly for agent clients.

## Project Flow Overview (Current Code)
- `src/mcp-server.ts` registers a single tool `deep-research` using `server.tool(...)`.
- Tool handler calls `research()` and `writeFinalReport()` from `src/deep-research.ts`.
- Caching via `LRUCache` in `mcp-server.ts` with key `JSON.stringify({ query, depth, breadth, existingLearnings })`.
- Transport: `StdioServerTransport(process.stdin, process.stdout)` with `server.connect(transport)`. This matches current code. Using `new StdioServerTransport()` without args is also valid per SDK docs.
- Logging: helper `log()` writes to stderr; progress notifications are logged only (no custom notifications emitted).
- Current search provider in orchestrator: Firecrawl (migration to Exa planned; see Notes).

## Agents & Tools Summary
- Tools exposed: one MCP tool `deep-research`.
- Orchestrator: `research(options)` in `src/deep-research.ts` (batches Gemini via `generateBatch()`; uses text splitters; composes final via `writeFinalReport()`).
- Feedback helper: `generateFeedback()` in `src/feedback.ts` (not exposed as MCP tool by default).

## Server & Transport
- **Instantiate server** with name/version and optional debounced notifications:
  ```ts
  import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
  import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

  const server = new McpServer(
    { name: 'deep-research', version: '1.0.0' },
    {
      // Reduce message spam during rapid list changes
      debouncedNotificationMethods: [
        'notifications/tools/list_changed',
        'notifications/resources/list_changed',
        'notifications/prompts/list_changed'
      ]
    }
  );

  // Current code passes streams explicitly; either form is acceptable
  const transport = new StdioServerTransport(process.stdin, process.stdout);
  await server.connect(transport);
  ```
  - SDK also supports `new StdioServerTransport()` without args. Current code uses explicit streams; both are valid.
  - Prefer `await server.connect(transport)` (async) vs promise chaining for clarity.

## Tools (deep-research)
- **Register tool** with zod input schema; return content in MCP format:
  ```ts
  import { z } from 'zod';

  // Current SDK and code use server.tool(...):
  server.tool(
    'deep-research',
    'Perform deep research on a topic using AI-powered web search',
    {
      query: z.string().min(1),
      depth: z.number().min(1).max(5),
      breadth: z.number().min(1).max(5),
      existingLearnings: z.array(z.string()).optional()
    },
    async ({ query, depth = 3, breadth = 3, existingLearnings = [] }) => {
      const result = await research({ query, depth, breadth, existingLearnings });
      const report = await writeFinalReport({ prompt: query, learnings: result.learnings, visitedUrls: result.visitedUrls });
      return {
        content: [{ type: 'text', text: report }],
        metadata: {
          learnings: result.learnings,
          visitedUrls: result.visitedUrls,
          stats: { totalLearnings: result.learnings.length, totalSources: result.visitedUrls.length }
        }
      };
    }
  );
  ```
  - Some SDK versions expose `server.registerTool(...)`. Use whichever matches your installed version; current code uses `server.tool(...)`.
  - For large outputs, consider returning `resource_link` items pointing to saved reports/resources instead of huge text blobs.

## Resources (optional, recommended)
- Expose read-only artifacts as **resources** to avoid large payloads and allow lazy fetch by clients:
  ```ts
  import { ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js';

  server.registerResource(
    'final-report',
    new ResourceTemplate('report://{hash}', { list: undefined }),
    { title: 'Deep Research Report', description: 'Generated final report', mimeType: 'text/markdown' },
    async (uri, { hash }) => ({ contents: [{ uri: uri.href, text: await loadReportByHash(hash) }] })
  );
  ```

## Notifications & Dynamics
- **Avoid manual internal access** like `server.server...`; use SDK APIs only.
- Prefer dynamic enable/disable/update of tools/resources to automatically emit `list_changed` notifications.
- Enable **debounced notifications** in the constructor to coalesce rapid changes.
- For progress updates:
  - Keep progress logging server-side (`stderr`).
  - If a client needs live progress, prefer exposing a status resource (e.g., `status://{taskId}`) that clients can poll. Avoid custom ad-hoc notifications unless you control both ends.

## Error Handling
- Catch errors and return a clear text message in `content`. Example:
  ```ts
  return { content: [{ type: 'text', text: `Error during deep research: ${message}` }] };
  ```
- Alternatively, throw and let the SDK/transport surface a JSON-RPC error. Be consistent.
- Log errors to `stderr` with structured context; never log secrets.

## Caching & Determinism
- Maintain a **stable cache key** (e.g., `JSON.stringify({query, depth, breadth, existingLearnings})`).
- Handle cache get/set errors gracefully; continue without cache when needed.
- Keep mapping deterministic `(query → results → contents → report)` for reproducible runs.

## Agent/API Flow (Recommended)
- **Client → MCP Tool**: `deep-research` with `{ query, depth, breadth, existingLearnings }`.
- **Tool Handler → Orchestrator**: Call `research()` which:
  - Uses Firecrawl for search (current). Migration to Exa (`exa-js`) recommended; gate via env flag when added.
  - Splits text via `createResearchSplitter()`; batches LLM calls via `generateBatch()`.
  - Applies Gemini grounding (if enabled) for citations.
  - Synthesizes outline/sections/summary via structured output schemas.
- **Orchestrator → Report**: `writeFinalReport()` composes final Markdown + references.
- **Handler → Response**: Return `{ content: [{ type: 'text', text: report }], metadata: { ... } }` or `resource_link`s.

## Security & Deployment
- For HTTP deployments, use `StreamableHTTPServerTransport` and enable **DNS rebinding protection**; restrict `allowedHosts`/`allowedOrigins`.
- Close transports/servers on connection close; ensure no file descriptor leaks.
- Test with **MCP Inspector** during development.

## Lotus Wisdom Highlights
- **Upaya (skillful means)**: Use resources for large artifacts to keep tool responses light and composable.
- **Integrate**: Debounce notifications to reduce noise; align server behavior with orchestrator caching/batching.
- **Verify**: Favor schema-validated inputs and deterministic outputs for repeatability.

## Notes
- Current orchestrator still uses Firecrawl; Exa integration is planned but not yet wired. Keep docs and code consistent until migration is complete.
- Environment loaded from `.env.local` in `mcp-server.ts`; do not log secrets.
- Progress notifications are not emitted; only stderr logging is used.

## Logs
- `src/mcp-server.ts`: `log(...args)` writes to stderr. Connection and errors are logged around `server.connect(...)`.
- `src/output-manager.ts`: debounced stdout logs and progress rendering for CLI mode.
- `src/feedback.ts`: uses `OutputManager` for analysis and cache logs.

## References
- MCP TypeScript SDK (README): https://github.com/modelcontextprotocol/typescript-sdk
- Running via stdio & HTTP + debouncing: see README sections (stdio, Streamable HTTP, Notification Debouncing)
- MCP Inspector: https://github.com/modelcontextprotocol/inspector
- Exa SDK (exa-js): https://github.com/exa-labs/exa-js

---
> Source: [ssdeanx/deep-research-mcp-server](https://github.com/ssdeanx/deep-research-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->

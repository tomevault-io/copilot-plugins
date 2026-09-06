## infographic-mcp-app

> Standalone MCP server that renders AntV Infographic diagrams with interactive text-based DSL.

# Infographic MCP App Server

Standalone MCP server that renders AntV Infographic diagrams with interactive text-based DSL.

## Architecture

```
server.ts        → 2 tools (read_me, create_infographic_view) + resource + cheat sheet
main.ts          → HTTP (Streamable) + stdio transports
src/mcp-app.tsx  → React widget: Infographic rendering via @antv/infographic
src/global.css   → Styling + auto-resize
```

## Tools

### `read_me` (text tool, no UI)
Returns a cheat sheet with element format, color palettes, coordinate tips, and examples. The model should call this before `create_infographic_view`.

### `create_infographic_view` (UI tool)
Takes `elements` — a string in Infographic DSL syntax. The widget uses `@antv/infographic` to render the diagram.

## Key Design Decisions

### Infographic DSL — text-based syntax
The input is a structured text format parsed by `@antv/infographic`, not JSON. It describes visual elements like shapes, text, icons, and decorative elements with a simple DSL (Domain Specific Language).

**Why:** The Infographic library provides rich, hand-drawn style visualizations with built-in animations and a concise syntax for creating complex diagrams.

**Trade-off:** The DSL is specific to AntV Infographic and differs from standard Excalidraw JSON format.

### Library-based rendering
The widget uses `@antv/infographic` library for rendering. It provides:
- Hand-drawn aesthetic with sketchy strokes
- Built-in animation support
- Responsive layouts
- Clean API for programmatic diagram generation

### Auto-sizing
The container has no fixed height. SVG gets `width: 100%` + `height: auto` with the `viewBox` preserving aspect ratio, so height scales proportionally to content.

### Checkpoint System

Two-tier storage for diagram state persistence:

#### Architecture
1. **Server-side store** (primary): `CheckpointStore` interface with 2 implementations:
   - `MemoryCheckpointStore` — Vercel/dev fallback (in-memory Map, lost on cold start)
   - `RedisCheckpointStore` — Vercel with Upstash KV (persistent, 30-day TTL)
   - Factory: `createVercelStore()` picks Redis if env vars exist, else Memory

2. **Widget** (render only): The current implementation renders the Infographic directly without localStorage caching.

#### Flow
- `create_infographic_view` saves the DSL elements to checkpoint store, returns `checkpointId`
- Widget renders using the elements passed via `ontoolresult` and `ontoolinput`
- `checkpointId` is used to reference the saved diagram state

### Checkpoint ID Generation
- `checkpointId` uses `crypto.randomUUID()` truncated to 18 chars (collision-resistant, URL-safe)

## Build

```bash
npm install
npm run build
```

Build pipeline: `tsc --noEmit` → `vite build` (singlefile HTML) → `tsc -p tsconfig.server.json` → `bun build` (server + index).

## Running

```bash
# HTTP (Streamable) — default, stateless per-request
npm run serve          # or: bun --watch src/main.ts
# Starts on http://localhost:3001/mcp

# stdio — for Claude Desktop
node dist/index.js --stdio

# Dev mode (watch + serve)
npm run dev
```

## Claude Desktop config

```json
{
  "mcpServers": {
    "infographic": {
      "command": "node",
      "args": ["<path>/dist/index.js", "--stdio"]
    }
  }
}
```

## Rendering Pipeline

### Tool Input (`ontoolinput`)
1. The model sends DSL elements as a string
2. Server saves to checkpoint store, returns `checkpointId`
3. Widget receives elements via `ontoolinput` callback
4. `Infographic` instance renders to the container with 100% width/height

### Result Handling (`ontoolresult`)
1. After tool execution, structured result contains `checkpointId` and `elements`
2. Widget updates state with the saved elements

### Container Height Management
- Widget captures initial container dimensions via `app.getHostContext()`
- Listens to `onhostcontextchanged` for dimension updates
- Syncs `document.documentElement.style.height` and `body.style.height` to match container

## Key Libraries

- **@antv/infographic**: Core rendering library for text-based diagram DSL
- **react/react-dom**: Widget UI framework
- **@modelcontextprotocol/sdk**: MCP server SDK
- **@modelcontextprotocol/ext-apps**: MCP Apps extension

## Cheat Sheet Reference

The `server.ts` loads `src/prompt.md` as the cheat sheet (`RECALL_CHEAT_SHEET`), containing:
- DSL syntax and element types
- Color palettes and styling options
- Positioning and layout tips
- Common patterns and examples

## Debugging

### Dev workflow
1. Edit source files
2. `npm run build` (or `npm run dev` for watch mode)
3. Restart the server process (module cache means hot reload doesn't pick up `server.ts` changes for tool definitions)
4. In Claude Desktop: restart the MCP server connection

### Widget logging

Use `fsLog()` function in `mcp-app.tsx` for debugging:

```typescript
fsLog("Debug message: " + JSON.stringify(data));
```

Log output routes through `app.sendLog()` with logger "FS".

### Common issues
- **No diagram appears:** Check that `ontoolinput` is firing and elements string is valid Infographic DSL
- **Container height issues:** Verify container dimensions are being captured correctly
- **Build errors:** Ensure `INPUT=src/mcp-app.html` environment variable is set for vite build

## Gotchas

- The Infographic DSL is text-based, not JSON
- Container must have explicit dimensions for Infographic to render correctly
- The `cross-env` package is required for setting environment variables in npm scripts on Windows
- Bun is used for building the server (`bun build`), but Node can run the output

---
> Source: [VancySavoki/infographic-mcp-app](https://github.com/VancySavoki/infographic-mcp-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->

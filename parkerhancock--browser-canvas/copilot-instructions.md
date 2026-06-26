## browser-canvas

> Handles dynamic bundling of scope extensions:

# Browser Canvas Development Guide

## Project Structure

```
browser-canvas/
├── src/
│   ├── client.ts                # CanvasClient API for operations
│   ├── server/
│   │   ├── index.ts             # Bun + Hono server entry
│   │   ├── watcher.ts           # File system watcher (chokidar)
│   │   ├── websocket.ts         # WebSocket handler
│   │   ├── canvas-manager.ts    # Canvas instance state
│   │   ├── vanilla.ts           # Vanilla mode: bridge injection, detection
│   │   ├── component-loader.ts  # Reusable component loading
│   │   ├── scope-builder.ts     # Dynamic scope bundling
│   │   ├── tailwind-builder.ts  # Dynamic CSS building
│   │   ├── validators.ts        # ESLint, scope, Tailwind validation
│   │   ├── validation-status.ts # Validation state for status API
│   │   └── log.ts               # Unified log system
│   └── browser/
│       ├── index.html           # React canvas shell
│       ├── app.tsx              # React app with react-runner
│       ├── scope.ts             # Pre-bundled dependencies
│       ├── event-bridge.ts      # window.canvasEmit implementation
│       ├── use-canvas-state.ts  # Two-way state sync hook
│       ├── linter.ts            # axe-core, overflow, image validation
│       └── components/ui/       # shadcn components
├── vanilla/
│   └── base.css                 # CSS variables and utilities for vanilla mode
├── skills/
│   └── canvas/
│       ├── SKILL.md             # Claude instructions
│       ├── components/          # Reusable components (auto-loaded)
│       └── references/          # Component documentation
├── hooks/
│   ├── hooks.json               # PostToolUse hook config
│   └── canvas-validation.sh     # Validation feedback injection
├── dist/                        # Built output (gitignored)
├── server.sh                    # Start server script
└── package.json
```

## Key Architecture

### Dual-Mode Support

The server auto-detects canvas mode by filename:

| File | Mode | Stack |
|------|------|-------|
| `App.jsx` | React | shadcn/ui + Tailwind + React hooks |
| `index.html` | Vanilla | Pure HTML/CSS/JS, CSS variables |

Both modes share the same file protocol, WebSocket bridge, and API.

### File-Based Protocol

Claude interacts via filesystem for data:

| Claude Action | File Operation |
|---------------|----------------|
| Create React canvas | Write `App.jsx` to new folder |
| Create Vanilla canvas | Write `index.html` to new folder |
| Update canvas | Edit `App.jsx` or `index.html` |
| Read log | Read `_log.jsonl` (grep for type/severity/category) |
| Read/write state | Read/write `_state.json` |

### TypeScript API (Operations)

Use `CanvasClient` for operations:

```typescript
import { CanvasClient } from "browser-canvas"
const client = await CanvasClient.fromServerJson()

await client.screenshot("my-app")  // Take screenshot
await client.close("my-app")       // Close canvas
await client.getState("my-app")    // Get state (alternative to file)
await client.setState("my-app", { step: 2 })  // Set state
await client.getStatus("my-app")   // Get validation status
await client.health()              // Check server health
```

### Server ↔ Browser Protocol (WebSocket)

```typescript
// Server → Browser
{ type: "reload", code: string }  // React: includes code; Vanilla: code omitted
{ type: "screenshot-request" }

// Browser → Server
{ type: "event", event: string, data: unknown }
{ type: "screenshot", dataUrl: string }
{ type: "error", message: string }
{ type: "set-state", state: Record<string, unknown> }
```

### Vanilla Mode Architecture

Vanilla canvases serve `index.html` with an injected bridge script:

```
src/server/vanilla.ts
├── detectCanvasMode()      # Check for App.jsx vs index.html
├── readVanillaCanvas()     # Read index.html from disk
└── injectVanillaBridge()   # Inject WebSocket bridge into HTML
```

The bridge provides:
- `window.canvasEmit(event, data)` - Send events to log
- `window.canvasState(newState?)` - Get/set two-way state
- Auto-reconnecting WebSocket
- Hot-reload on file change (full page refresh)

Styling via `/base.css` route serving `vanilla/base.css` with CSS variables.

### PostToolUse Hook (Validation Feedback)

The plugin includes a PostToolUse hook that automatically injects validation errors after Write/Edit operations on `App.jsx` files:

```
hooks/
├── hooks.json               # Hook configuration
└── canvas-validation.sh     # Fetches validation status, formats output
```

Flow:
1. Claude writes/edits `App.jsx`
2. Server validates (ESLint, scope, Tailwind)
3. Hook calls `/api/canvas/:id/status?wait=true`
4. Validation errors injected via `additionalContext`

The status API supports waiting for pending validation:
```
GET /api/canvas/:id/status?wait=true&timeout=2000
```

## Development Commands

```bash
bun install          # Install dependencies
bun run dev          # Start dev server with watch
bun run build        # Build browser bundle
bun run typecheck    # Type check
```

## Adding shadcn Components

1. Copy component to `src/browser/components/ui/`
2. Export from `src/browser/scope.ts`
3. Document in `skills/canvas/references/components.md`
4. Rebuild: `bun run build`

## Adding Reusable Components

Reusable components are auto-loaded from multiple directories (later overrides earlier):

1. `skills/canvas/components/*.jsx` - Skill-level (ships with package)
2. `~/.claude/canvas/components/*.jsx` - Global user components
3. `.claude/canvas/components/*.jsx` - Project-level components

Component file format:

```jsx
/**
 * MyComponent - Brief description
 * @prop {string} title - Prop description
 * @prop {function} onClick - Callback description
 */
function MyComponent({ title, onClick }) {
  return (
    <Card>
      <CardHeader><CardTitle>{title}</CardTitle></CardHeader>
    </Card>
  )
}
```

Components are compiled with sucrase and sent to the browser via WebSocket. No imports needed - just use `<MyComponent />` in App.jsx.

## Project Extensions

Projects can extend browser-canvas by creating files in `.claude/canvas/`:

| File | Purpose | Rebuild Trigger |
|------|---------|-----------------|
| `scope.ts` | Add npm packages to scope | JS bundle |
| `tailwind.config.js` | Add Tailwind plugins | CSS |
| `styles.css` | Custom CSS rules | CSS |
| `components/*.jsx` | Project-specific components | Components |

### scope-builder.ts

Handles dynamic bundling of scope extensions:
- Reads base `scope.ts` and project `.claude/canvas/scope.ts`
- Converts relative imports to absolute paths
- Uses Bun plugin to intercept scope imports during build
- Merges base scope with project `extend` object

### tailwind-builder.ts

Handles dynamic CSS building:
- Generates merged Tailwind config from base + project
- Runs Tailwind CLI with merged config
- Appends custom CSS from `.claude/canvas/styles.css`

Extensions are detected at server startup. If found, rebuilds are triggered (~50ms).

## Canvas Directory (Runtime)

Default: `.claude/artifacts/` in current directory:

```
.claude/artifacts/
├── server.json              # Server state (port, active canvases)
├── _server.log              # Server logs
├── react-app/               # React mode canvas
│   ├── App.jsx              # React component code
│   ├── _log.jsonl           # Unified log (events, errors, validation)
│   ├── _state.json          # Two-way state
│   └── _screenshot.png      # Screenshot output (via API)
└── vanilla-app/             # Vanilla mode canvas
    ├── index.html           # HTML/CSS/JS code
    ├── _log.jsonl           # Same log format
    ├── _state.json          # Same state sync
    └── _screenshot.png      # Same screenshot API
```

## Testing

Manual testing:

**React Mode:**
1. Start server: `./server.sh`
2. Create test canvas: write to `.claude/artifacts/test/App.jsx`
3. Verify browser opens and renders
4. Test hot-reload by editing the file
5. Test events with `window.canvasEmit()`
6. Test screenshots via `CanvasClient.screenshot()`
7. Check `_log.jsonl` for validation notices

**Vanilla Mode:**
1. Create test canvas: write to `.claude/artifacts/vanilla-test/index.html`
2. Include `<link rel="stylesheet" href="/base.css">` for styling
3. Verify browser opens and renders
4. Test hot-reload (full page refresh on change)
5. Test `window.canvasEmit()` and `window.canvasState()`
6. Verify state sync via `_state.json`

## Code Style

- TypeScript for all server code
- JSX/TSX for browser code
- Tailwind for styling
- No semicolons (Bun default)
- 2-space indentation

## Key Dependencies

- **bun** - Runtime and bundler
- **hono** - HTTP framework
- **chokidar** - File watching
- **react-runner** - JSX execution
- **sucrase** - JSX compilation for components
- **html2canvas** - Screenshots
- **eslint** - Server-side code validation
- **axe-core** - Browser-side accessibility/visual validation
- **@radix-ui** - Primitives for shadcn
- **recharts** - Charts
- **lucide-react** - Icons
- **tailwindcss** - Styling

---
> Source: [parkerhancock/browser-canvas](https://github.com/parkerhancock/browser-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->

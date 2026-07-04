## handsfree

> HandsFree is a macOS desktop assistant that controls the user's computer through voice and text. It combines a voice conversation layer with a visual computer-use layer.

# HandsFree — Agent Architecture

HandsFree is a macOS desktop assistant that controls the user's computer through voice and text. It combines a voice conversation layer with a visual computer-use layer.

## Architecture

```
User (voice/text)
  │
  ▼
Realtime (gpt-realtime-2)          ← voice model, handles conversation
  │
  ├─ Instant tools                 ← CGEvent keyboard, clipboard, app launch
  │   type_text, press_key,
  │   launch_app, activate_app,
  │   clipboard_read/write,
  │   show_grid, click_grid,
  │   open_url, list_apps
  │
  ├─ use_computer(task)            ← delegates to CUA
  │   │
  │   ▼
  │  CUA Runner (Electron main)    ← Responses API loop
  │   │
  │   ├─ gpt-5.5 + { type: "computer" }
  │   │   sees screenshots, returns click/type/scroll actions
  │   │
  │   └─ Swift MCP (HandsFreeComputerUse)
  │       cua_screenshot, cua_click, cua_type, cua_keypress, etc.
  │       executes actions via native macOS APIs (CGEvent, AX, NSWorkspace)
  │
  ├─ stop_computer                 ← abort running CUA task
  │
  └─ MCP connectors                ← openwork-ui, chrome-devtools, custom
```

## Three Levels of Control

| Level | Speed | Tools | When to use |
|-------|-------|-------|-------------|
| **Instant** | <100ms | type_text, press_key, click_grid, clipboard, launch_app | Direct keyboard/mouse, grid clicks, app switching |
| **Grid** | <200ms | show_grid, click_grid | User says "click C2" — maps zone to coordinates |
| **CUA** | 5–45s | use_computer | Multi-step visual tasks: navigate UI, find elements, fill forms |

## Tool Selection Rules

The Realtime model (gpt-realtime-2) follows these rules:

- "Type X" → `type_text` (instant, literal)
- "Press enter" / "command+k" → `press_key` (instant)
- "Click C2" → `click_grid` (instant, grid overlay)
- "Open Slack" → `launch_app` (instant)
- "Open Slack and message Omar" → `use_computer` (needs vision)
- "Stop" / "cancel" → `stop_computer` (instant)

**Key principle:** the Realtime model cannot see the screen. It must delegate all visual tasks to `use_computer`. Typing and key presses are always instant — never routed through CUA.

## Swift MCP Server (HandsFreeComputerUse)

The Swift binary runs as a stdio MCP server. Electron spawns it on launch.

It holds macOS permissions (Accessibility, Screen Recording) and executes:
- Screenshots via CGWindowListCreateImage
- Clicks/keyboard/scroll via CGEvent (InputService)
- AX tree queries via Accessibility APIs (AccessibilityService)
- App management via NSWorkspace
- Clipboard via NSPasteboard
- Grid overlay via GridOverlayWindow

The Swift process can restart independently when permissions change without restarting Electron.

## CUA Runner

The CUA runner in `electron/main.mjs` implements the OpenAI Responses API computer-use loop:

1. Send task + tools to gpt-5.5
2. Model returns `computer_call` with `actions[]`
3. Execute each action via Swift MCP (`cua_click`, `cua_type`, etc.)
4. Take screenshot via `cua_screenshot`
5. Send screenshot back as `computer_call_output`
6. Repeat until model stops returning `computer_call`

Screenshots are PNG at logical display resolution (points, not retina pixels) so model coordinates map 1:1 to screen coordinates.

## Grid Overlay

6 columns (A–F) × 4 rows (1–4) = 24 zones. Subtle, click-through, always-on-top.

User says "click C2" → `click_grid` resolves to center of zone → instant CGEvent click with agent cursor animation.

## Evals

See `test/cua-eval.mjs` for the eval suite. See below for how to run and write evals.

### Running evals

```bash
# 1. Start HandsFree
pnpm run dev

# 2. Open the test page in Chrome
open -a "Google Chrome" test/cua-test.html

# 3. Run all evals
node test/cua-eval.mjs

# 4. Run a single eval
node test/cua-eval.mjs --only=click-red
```

### What evals test

Evals cover three areas:

**1. Instant tools** — verify direct tool calls work without CUA:
- `type_text`: types exact text via CGEvent
- `press_key`: presses key combos
- `click_grid`: clicks grid zones by label
- `clipboard_read/write`: clipboard round-trip
- `show_grid/hide_grid`: overlay toggle
- `list_apps`, `display_info`, `check_permissions`: system queries

**2. CUA visual tasks** — verify gpt-5.5 can see and interact with UI:
- Click large, medium, and small targets
- Type into form fields
- Navigate tabs and sidebar
- Scroll and find elements
- Multi-step workflows (click + type + navigate)

**3. Behavioral tests** (manual, via voice):
- "Type hello" uses `type_text`, not `use_computer`
- "Press enter" uses `press_key`, not `use_computer`
- "Stop" during CUA calls `stop_computer` immediately
- Model speaks English regardless of accent
- Model doesn't respond to background noise
- Preambles are short and action-descriptive
- Model doesn't paraphrase when typing — types literally

### Writing new evals

Add tests to `test/cua-eval.mjs`:

```js
{
  id: "my-test",
  name: "CUA: description of what we're testing",
  type: "cua",  // or "direct" for instant tools
  task: "Plain language task for the CUA model",
  maxTurns: 5,  // fail if CUA takes more turns
  maxTime: 30,  // fail if slower than 30 seconds
  verify: "document.querySelector('#result')?.innerText",  // DOM check
  expect: (v) => v?.includes("expected value"),
}
```

For instant tool tests:
```js
{
  id: "my-direct-test",
  name: "tool_name: what it does",
  type: "direct",
  tool: "tool_name",
  args: { key: "value" },
  expect: (v) => v?.ok === true,
}
```

### Test page

`test/cua-test.html` provides varied targets:
- **Easy**: 4 large colored buttons
- **Medium**: 8 standard buttons, tab bar, sidebar nav
- **Hard**: 10 tiny dots, 6 emoji icons
- **Forms**: text, email, search, select, textarea, checkboxes, radios
- **Scroll**: 12-item scroll area with hidden target
- **Drag**: draggable item + drop zone
- **Dropdown**: click-to-open menu with 5 items
- Event log at the bottom captures all interactions

### Eval metrics

For CUA tests, track:
- **Pass rate**: does the task succeed?
- **Turns**: how many API round-trips?
- **Time**: total wall-clock time?
- **Precision**: did it click the right target (check DOM event log)?

For instant tools, track:
- **Pass rate**: does the tool return ok?
- **Correctness**: does the DOM/clipboard/system state match?

### CI considerations

CUA evals require a running macOS display, Accessibility permission, and an OpenAI API key. They are not suitable for headless CI. Run them locally before merging.

Instant tool evals (grid, clipboard, display_info, etc.) could run in CI if the Swift MCP server is available, but currently require the full Electron app.

## Prompt Design

The Realtime instructions follow the [OpenAI Realtime 2 Prompting Guide](https://platform.openai.com/docs/guides/realtime-model-capabilities).

Key sections:
- **Role**: macOS computer-control voice assistant
- **Language**: English default, accent-immune
- **Reasoning**: minimal for instant tools, brief for CUA delegation
- **Preambles**: one short sentence before CUA, skip for instant actions
- **Tool rules**: explicit selection logic (type → type_text, visual → use_computer)
- **Safety**: literal typing, confirm before sending messages, no destructive combos

## File Map

```
electron/main.ts           Electron bootstrap: Realtime session, CUA runner, IPC handlers, tool definitions
electron/preload.ts        Typed IPC bridge to renderer
electron/main/mcp-env.ts   Child-process PATH enrichment for stdio MCPs
electron/main/native-helper.ts  Computer Use helper path resolution
dist-main/                 Compiled Electron main/preload output (generated)
src/                        React UI (timeline, composer, connectors, shader orb)
native/HandsFree/
  Sources/ComputerUse/
    MCPServer.swift          MCP JSON-RPC server, tool dispatch
    AccessibilityService.swift  AX tree, screenshots
    InputService.swift       CGEvent clicks, keyboard, scroll, drag
    GridOverlayWindow.swift  Screen grid overlay
    AgentCursorWindow.swift  Visual cursor animation
    Types.swift              AXElement, ScreenshotMetadata, AppSnapshot
test/
  cua-test.html             Click/type/scroll test targets
  cua-eval.mjs              Automated eval script
```

---
> Source: [different-ai/handsfree](https://github.com/different-ai/handsfree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->

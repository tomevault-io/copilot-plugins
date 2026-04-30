## ghost-os

> Accessibility-first perception and execution engine that gives AI agents the

# Ghost OS -- Developer Guide

Accessibility-first perception and execution engine that gives AI agents the
ability to see, understand, and operate any macOS application through the
accessibility tree and visual perception.

## Build & Run

```bash
swift build                          # Requires Swift 6.2+ (swiftly), macOS 14+
.build/debug/ghost mcp               # Start MCP server (stdio)
.build/debug/ghost setup             # Interactive first-run wizard
.build/debug/ghost doctor            # Diagnostic health check
.build/debug/ghost status            # Quick status
.build/debug/ghost version           # Version info
```

Release: `scripts/build-release.sh` builds tarball, verifies 3-way version
consistency (Types.swift + server.py + ghost-vision), and codesigns the binary.

## Architecture

MCP-first, single-threaded synchronous. No persistent daemon, no socket IPC, no
async state model. Every MCP tool call queries the AX tree fresh.

```
AI Agent (Claude Code / Cursor / any MCP client)
    |
    | MCP Protocol (stdio, auto-detects Content-Length vs NDJSON)
    |
ghost mcp (Swift binary, @MainActor)
    |
    +-- Perception (7 tools)  -- AX tree via AXorcist
    +-- Annotate (1 tool)     -- Set-of-Marks labeled screenshots, zero ML
    +-- Actions (10 tools)    -- AX-native -> CDP -> VLM -> synthetic cascade
    +-- Wait (1 tool)         -- 6 polling condition types
    +-- Recipes (5 tools)     -- JSON workflow storage and execution
    +-- Vision (2 tools)      -- ShowUI-2B VLM via Python sidecar
    +-- Learning (3 tools)    -- CGEvent tap recording for recipe synthesis
    |
    +-- AXorcist (github.com/steipete/AXorcist, from: "0.1.0")
    +-- ScreenCaptureKit (linked framework)
```

The learning subsystem is the one exception to single-threaded: CGEvent tap runs
on a dedicated nonisolated background Thread with os_unfair_lock and CFRunLoop.

**stdout is sacred.** All MCP protocol data flows through stdout. A single stray
`print()` or `Swift.print()` will corrupt the protocol and break the connection.
All logging goes through `Log` (writes to stderr).

## Key Files

| File | What It Does |
|------|-------------|
| `Sources/GhostOS/MCP/MCPTools.swift` | All 29 tool definitions (the agent-facing contract) |
| `Sources/GhostOS/MCP/MCPDispatch.swift` | Tool routing and response formatting |
| `Sources/GhostOS/Perception/Perception.swift` | AX tree reading, semantic depth tunneling |
| `Sources/GhostOS/Actions/Actions.swift` | Multi-strategy click/type (AX -> CDP -> VLM -> synthetic) |
| `Sources/GhostOS/Recipes/RecipeEngine.swift` | Step-by-step execution with preconditions and wait conditions |
| `Sources/GhostOS/Learning/LearningRecorder.swift` | CGEvent tap + AX enrichment for recording user actions |
| `Sources/GhostOS/Vision/VisionBridge.swift` | ShowUI-2B VLM grounding via Python sidecar |
| `Sources/GhostOS/Vision/VisionPerception.swift` | ghost_ground (works) and ghost_parse_screen (stub, YOLO not implemented) |
| `Sources/GhostOS/Perception/Annotate.swift` | Set-of-Marks annotation (AX-powered, zero ML) |
| `Sources/GhostOS/Screenshot/ScreenCapture.swift` | Window screenshots via ScreenCaptureKit (captures background windows) |
| `GHOST-MCP.md` | 13 rules for agents using Ghost OS tools + recipe JSON schema |

## 29 MCP Tools

| Category | Tools |
|----------|-------|
| Perception (7) | context, state, find, read, inspect, element_at, screenshot |
| Annotate (1) | annotate |
| Actions (10) | click, type, press, hotkey, scroll, hover, long_press, drag, focus, window |
| Wait (1) | wait (urlContains, titleContains, elementExists, elementGone, urlEquals, titleEquals) |
| Recipes (5) | recipes, run, recipe_show, recipe_save, recipe_delete |
| Vision (2) | ground, parse_screen |
| Learning (3) | learn_start, learn_stop, learn_status |

**Note:** `ghost_ground` sends a description to the ShowUI-2B vision model and
returns click coordinates. `ghost_parse_screen` is a stub (the YOLO detection
endpoint is not yet implemented). For detecting all interactive elements without
ML, use `ghost_annotate` instead (AX-powered Set-of-Marks).

## AXorcist APIs We Use

Ghost OS depends on AXorcist for all accessibility tree access. Check
AXorcist source at `../AXorcist/Sources/AXorcist/` before building new features.

| API | Purpose |
|-----|---------|
| `Element.children()` | Walk element tree (checks 14+ alternative attributes) |
| `Element.title()`, `.descriptionText()`, `.value()` | Read element text |
| `Element.visibleCharacterRange()` + `.string(forRange:)` | Web content text |
| `Element.numberOfCharacters()` | Text length for range-based extraction |
| `Element.role()`, `.position()`, `.size()` | Element metadata |
| `Element.performAction(.press)` | Click elements via AX |
| `Element.typeText(text, delay:)` | Reliable typing with configurable delay |
| `Element.url()` | Read AXURL from web areas (page URL) |
| `Element.isEditable()` | Check if focused element is a text field |
| `InputDriver.click()`, `.tapKey()`, `.hotkey()` | Synthetic input |
| `AXObserverCenter` | Real-time AX notifications |
| `AccessibilityPermissions` | Permission checking |

## Code Conventions

- Swift 6.2 strict concurrency. @MainActor default isolation.
- All logging via `Log` (stderr). Never use print (corrupts MCP stdout).
- No force unwraps except in tests.
- Error responses include a `suggestion` field telling the agent what to do next.
- No app-specific hacks. All fixes must be generic.
- Version must match in 3 places: `Types.swift`, `server.py`, `ghost-vision`.
- Semantic depth tunneling: empty layout containers (AXGroup with no content)
  cost 0 depth. Budget of 25 reaches content at 30+ DOM levels.

## Technical Gotchas

- **Chrome AXStaticText**: AXorcist's typed accessor returns nil; must use raw
  AXUIElementCopyAttributeValue to get AXValue from Chrome text elements.
- **Modifier clearing**: After every hotkey, post CGEvent flagsChanged with empty
  flags to prevent stuck modifier keys.
- **Focus management**: Perception tools work from background. Click/type auto-save
  and restore focus. Press/hotkey/scroll/hover/long_press/drag leave target focused.
- **Transport detection**: First byte determines Content-Length (Claude Code) vs
  NDJSON (Claude Desktop) protocol.
- **Native apps**: Only show menu bar when backgrounded; need focus for full AX tree.
  Messages and Finder are exceptions (work backgrounded).
- **Chrome tabs**: Invisible to AX API (Chromium design choice). Use ctrl+tab to cycle.
- **Vision sidecar fragility**: Python dependency management has caused 3 consecutive
  patch releases (issues #1, #3, #7). The sidecar requires pinned versions of
  mlx-vlm, transformers, and mlx-lm. Always test `ghost doctor` after touching
  vision-sidecar/ and verify on a fresh venv.

## Testing

```bash
swift test                           # Runs LocatorBuilderTests
```

E2E testing is manual: run `ghost mcp`, send JSON-RPC messages, verify responses.
Recipes should be tested 3+ times before bundling.

## Release Checklist

1. Bump version in: `Types.swift` + `server.py` + `ghost-vision` (must match)
2. Update README: tool count, tarball URL, What's New section
3. Update GHOST-MCP.md if new tools or schema changes
4. `scripts/build-release.sh` produces tarball + SHA256
5. `gh release create v{VERSION} {tarball} --title "Ghost OS v{VERSION}"`
6. Update Homebrew tap Formula/ghost-os.rb: url + sha256 + version
7. Manual install test: extract tarball, verify binary runs, ghost doctor
8. Verify installed version works end-to-end

## Contributing

- Recipes are the easiest contribution: JSON files in `recipes/`. See GHOST-MCP.md
  for the full recipe JSON schema (10 action types, parameters, wait conditions).
  Test each recipe 3+ times before submitting.
- Check AXorcist source before building new features. It may already have what you need.
- PRs for all code changes. Include what you tested and how.

---
> Source: [ghostwright/ghost-os](https://github.com/ghostwright/ghost-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->

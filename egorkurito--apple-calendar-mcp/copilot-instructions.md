## apple-calendar-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Apple Calendar MCP server — an MCP server for Apple Calendar via the native EventKit API. Solves the problem of incorrect recurring event handling in the AppleScript approach. Two-layer architecture: TypeScript MCP server calls a Swift CLI binary (`apple-bridge`) via execa.

## Build & Run

```bash
# Full build (Swift + TypeScript)
./scripts/build.sh

# Swift bridge only
cd swift && swift build -c release && codesign --force --sign - --entitlements apple-bridge.entitlements .build/release/apple-bridge

# TypeScript only
npm run build    # tsc → build/

# Verify Swift binary
swift/.build/release/apple-bridge doctor
swift/.build/release/apple-bridge calendars
swift/.build/release/apple-bridge today

# Test MCP server (interactive)
APPLE_BRIDGE_BIN=swift/.build/release/apple-bridge npx @anthropic-ai/sdk mcp-run build/index.js
```

No tests. Verification is manual via CLI commands and MCP inspector.

## Architecture

```
Claude Code ──stdio JSON-RPC──▶ TypeScript MCP Server ──execa──▶ Swift CLI (apple-bridge) ──EventKit──▶ macOS Calendar
```

**TypeScript layer** (`src/`):
- `index.ts` — McpServer with StdioServerTransport, reads `APPLE_BRIDGE_BIN` env var
- `bridge/swift.ts` — SwiftBridge class: calls binary via execa, parses JSON envelope `{status, data, error}`, 30s timeout
- `tools/calendar.ts` — 6 MCP tools with Zod validation: get_calendars, get_events, get_today_events, search_events, create_event, delete_event

**Swift layer** (`swift/Sources/AppleBridge/`):
- `main.swift` — ArgumentParser CLI with 6 subcommands (mirrors MCP tools)
- `CalendarService.swift` — EventKit wrapper; `predicateForEvents` expands recurring events; DispatchSemaphore for sync TCC permission request; `#available(macOS 14.0, *)` for compatibility
- `Models.swift` — Codable DTOs, BridgeOutput<T> JSON envelope, ISO8601 parsing (two formats: with/without fractional seconds), BridgeError enum with LocalizedError

**Key detail**: `search_events` filters on the TypeScript side (client-side), not in Swift.

## Important Conventions

- **stdout is reserved for MCP JSON-RPC** — in TypeScript use only `console.error()` for logs
- **Swift CLI always returns JSON envelope**: `{"status":"ok","data":...}` or `{"status":"error","error":"..."}`
- **ESM modules**: package.json `"type": "module"`, imports with `.js` extension
- **Swift**: `-parse-as-library` flag in Package.swift is required for @main + ArgumentParser
- **TCC permissions**: binary must be signed with entitlements (`apple-bridge.entitlements`) for calendar access
- **APPLE_BRIDGE_BIN** — the only way to pass the Swift binary path to the MCP server

## MCP Integration

Via `.mcp.json` in a consumer project or `claude mcp add --scope user`:
```json
{
  "mcpServers": {
    "apple-calendar": {
      "command": "node",
      "args": ["/path/to/build/index.js"],
      "env": { "APPLE_BRIDGE_BIN": "/path/to/swift/.build/release/apple-bridge" }
    }
  }
}
```

## Troubleshooting

- **Calendar access denied**: `tccutil reset Calendar` → re-run `apple-bridge doctor` for a new TCC prompt
- **Recurring events not expanding**: ensure `predicateForEvents` is being used, not direct `EKEvent` access
- **ISO8601 parse error**: Swift expects format `2026-02-07T10:00:00Z` or `2026-02-07T10:00:00.000Z`

---
> Source: [EgorKurito/apple-calendar-mcp](https://github.com/EgorKurito/apple-calendar-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

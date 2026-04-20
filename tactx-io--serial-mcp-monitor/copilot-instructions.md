## serial-mcp-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run compile       # Compile TypeScript to out/
npm run watch         # Watch mode compilation
npm run lint          # ESLint on src/
npm run test          # Compile + lint + run vscode-test
```

To run the extension during development, use F5 in VSCode (launches Extension Host via `.vscode/launch.json`).

## Architecture

This is a VSCode extension that bridges serial ports to an MCP (Model Context Protocol) server, enabling AI agents to interact with hardware.

**Single source file:** [src/extension.ts](src/extension.ts) contains the entire extension.

**Three integrated components, all initialized in `activate()`:**

1. **Webview Panel** — Opens a live traffic monitor (`Serial MCP Live Monitor`) in VSCode's second column, showing RX/TX/MCP messages color-coded via `postMessage`.

2. **MCP Server** (`@modelcontextprotocol/sdk`) — Exposes two tools:
   - `list_ports`: Lists available serial ports via `SerialPort.list()`
   - `write_serial`: Writes text to the currently open serial connection

3. **TCP Bridge** on `localhost:8888` — Accepts newline-delimited JSON-RPC from external agents (e.g. Claude Desktop). Connects each TCP socket as a custom MCP transport.

**Serial connection** is held in module-level `serialConn: SerialPort | null`. It's opened via the `serial-mcp-monitor.connect` command (QuickPick port selector, hardcoded 115200 baud). Incoming data is forwarded to the webview as RX messages.

**Output:** TypeScript compiles to `out/` (Node16 modules, ES2022 target).

## Key constraints

- The TCP bridge transport implementation in `extension.ts` is a manual duck-typed object — the MCP SDK's `StdioTransport` is not used here.
- The extension activates on `onStartupFinished` and immediately creates the webview panel (not on-demand via the registered command).
- `package.json` has `"type": "module"` — imports use `.js` extensions (see MCP SDK imports).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tactx-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

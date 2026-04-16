## pocket-agent

> This project is a VS Code extension called "Pocket Agent" that provides a mobile interface for Cursor's composer and a web interface to view the captured chat.

# Project Overview

This project is a VS Code extension called "Pocket Agent" that provides a mobile interface for Cursor's composer and a web interface to view the captured chat.

## Key Components

-   **VS Code Extension**: [`src/cursor-plugin.js`](mdc:src/cursor-plugin.js) - Main entry point for the extension. Captures chat data from Cursor.
-   **Express Backend Server**: [`src/server/index.js`](mdc:src/server/index.js) - Receives chat data, serves a web view of the chat, and handles WebSocket connections.
-   **Package Definition**: [`package.json`](mdc:package.json) - Extension metadata and dependencies.
-   **Chat HTML Snapshot (Example)**: [`html`](mdc:html) - An example of the HTML structure captured from Cursor's chat panel, used for styling reference.

## Core Functionality

### Extension ([`src/cursor-plugin.js`](mdc:src/cursor-plugin.js))
1.  **CDP Interaction**: Connects to Cursor's debug port (default: `9223`, configurable via `CURSOR_DEBUG_PORT`) using Chrome DevTools Protocol.
2.  **Chat Content Extraction**: Periodically (default: `5000ms`, configurable via `POCKET_AGENT_POLL_INTERVAL`) polls Cursor's chat panel, specifically extracting the `outerHTML` of the `div.pane-body div.conversations` element.
3.  **Data Transmission**: Sends the extracted HTML content as a JSON payload to the `/chat-update` endpoint of the backend server.
4.  **Logging**: Provides output logging to both the console and a dedicated "Pocket Agent" VS Code output channel.

### Server ([`src/server/index.js`](mdc:src/server/index.js))
1.  **Data Reception**: Receives chat HTML from the extension at `/chat-update` and stores the latest version in memory.
2.  **Chat Web View**:
    *   Serves a page at `/view-chat` that displays the captured chat content.
    *   Uses an iframe pointing to `/get-chat-html`.
    *   The `/get-chat-html` route injects custom CSS styles (mimicking Cursor's UI) along with the chat HTML. This styling approach is detailed in [`chat-styling-strategy.mdc`](mdc:.cursor/rules/chat-styling-strategy.mdc).
3.  **WebSocket Server**: Implements a Socket.IO server for potential real-time communication with other clients (e.g., a mobile app).
4.  **Static Files**: Serves static client assets from [`src/client/`](mdc:src/client).

## Configuration (Environment Variables for Extension)
-   `POCKET_AGENT_SERVER_URL`: Backend server URL (default: `http://localhost:3300`).
-   `CURSOR_DEBUG_PORT`: Debug port for CDP (default: `9223`).
-   `POCKET_AGENT_POLL_INTERVAL`: Interval for polling chat updates (default: `5000ms`).

## Related Workflows
-   Extension data capture: [`extension-workflow.mdc`](mdc:.cursor/rules/extension-workflow.mdc)
-   Server data handling and view: [`server-workflow.mdc`](mdc:.cursor/rules/server-workflow.mdc)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lacymorrow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

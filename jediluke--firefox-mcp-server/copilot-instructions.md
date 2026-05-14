## firefox-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Server
- `npm start` - Run the MCP server
- `npm run dev` - Run server with Node.js debugger

### Testing
- `npm test` - Run basic Firefox functionality test
- `npm run install-firefox` - Install Firefox browser for Playwright

### Example Scripts
- `npm run example:basic` - Run basic demo script
- `npm run example:multiplayer` - Run multiplayer testing demo

## Active Implementation

The MCP server implementation is in `index.js`.

The server provides:
- **Class**: `FirefoxMCPServer` (version 1.0.0)
- **29 MCP tools** for browser automation and debugging
- **Multi-session support** with complete isolation
- **Advanced debugging capabilities** (console logs, WebSocket monitoring, network activity, performance metrics)

## Architecture Overview

### Core Components
1. **MCP Server**: Uses `@modelcontextprotocol/sdk` for stdio communication
2. **Browser Engine**: Playwright Firefox automation
3. **Session Management**: Map-based storage for browser contexts and pages
4. **Debug Buffers**: Per-tab in-memory storage for monitoring data

### Key Data Structures
- `this.browser` - Main Firefox browser instance
- `this.contexts` - Map of contextId → BrowserContext (for session isolation)
- `this.pages` - Map of tabId → Page (for tab management)
- `this.consoleLogs` - Map of tabId → array of console log events
- `this.jsErrors` - Map of tabId → array of JavaScript error events
- `this.networkActivity` - Map of tabId → array of network events
- `this.wsMessages` - Map of tabId → array of WebSocket messages
- `this.performanceMetrics` - Map of tabId → performance data
- `this.activeTabId` - Currently active tab identifier

### Session Isolation Architecture
Each session gets its own browser context with:
- Separate cookies and storage
- Independent session data
- Isolated network monitoring
- Individual debug buffers

## Tool Categories

### Browser Management (2 tools)
- `browser_launch` - Launch Firefox with multi-session support and debugging
- `browser_close` - Close browser and clean up all resources

### Session Management (4 tools)
- `session_create` - Create isolated browser session with separate cookies/storage
- `session_list` - List all active sessions with URLs and status
- `session_close` - Close specific session and free resources
- `session_set_active` - Set default session for subsequent operations

### Page Navigation (5 tools)
- `page_navigate` - Navigate to any URL
- `page_reload` - Refresh current page
- `history_back` - Navigate back in browser history
- `history_forward` - Navigate forward in browser history
- `url_get_current` - Get current page URL

### Element Interaction (5 tools)
- `element_click` - Click elements by selector or coordinates
- `element_drag` - Drag and drop with smooth animations
- `element_wait` - Wait for elements to appear or become visible
- `input_type` - Type text into input fields
- `keyboard_press` - Send keyboard events with modifier support

### Content Extraction (3 tools)
- `html_extract` - Extract HTML content from page or elements
- `text_extract` - Get visible text content
- `page_screenshot` - Capture screenshots

### JavaScript Execution (1 tool)
- `javascript_execute` - Run custom JavaScript in browser context

### Advanced Debugging (9 tools)
- `debug_console_logs` - Browser console output (log, error, warn, info, debug)
- `debug_javascript_errors` - JS errors with full stack traces
- `debug_network_activity` - HTTP request/response tracking
- `debug_websocket_messages` - WebSocket traffic (perfect for Phoenix LiveView)
- `debug_performance_metrics` - Timing, memory usage, paint events
- `debug_activity_all` - Combined feed of all debug events
- `debug_helpers_inject` - Inject custom debugging utilities
- `debug_monitoring_start` - Start/restart comprehensive monitoring
- `debug_buffers_clear` - Clear accumulated debug data

## Usage Patterns

### Multi-Session Setup for Multiplayer Testing
```javascript
// Launch and create isolated sessions
browser_launch({headless: false})
session_create({sessionId: "player1", url: "http://localhost:4000/game"})
session_create({sessionId: "player2", url: "http://localhost:4000/game"})

// Interact with specific sessions
element_click({sessionId: "player1", selector: "button.create-game"})
element_click({sessionId: "player2", selector: "button.join-game"})
```

### Phoenix LiveView Debugging
```javascript
// Monitor WebSocket traffic for LiveView
debug_websocket_messages({sessionId: "player1", limit: 10})

// Get console logs with error filtering
debug_console_logs({sessionId: "player1", types: ["error", "warn"]})

// Combined debug feed
debug_activity_all({sessionId: "player1", limit: 50})
```

### Enhanced Interactions
```javascript
// Keyboard events for sliders/controls
keyboard_press({
  selector: ".slider-forward",
  key: "ArrowLeft",
  repeat: 5
})

// Drag operations for UI elements
element_drag({
  selector: ".draggable-item",
  offsetX: -100,
  duration: 500,
  steps: 10
})
```

## Development Notes

### File Structure
- Main implementation: `index.js`
- Documentation: `README.md`, `docs/README-multi.md`, `docs/ENHANCEMENT_USAGE.md`
- Examples: `examples/demo-tibia.js`, `examples/demo-tibia-simple.js`, `examples/demo-multiplayer.js`
- Tests: `tests/test.js`

### Dependencies
- `@modelcontextprotocol/sdk@^0.5.0` - MCP protocol implementation
- `playwright@^1.40.0` - Browser automation

### Configuration
Uses stdio transport for MCP communication. Typical MCP client configuration:
```json
{
  "mcpServers": {
    "firefox-control": {
      "command": "node",
      "args": ["index.js"],
      "cwd": "/path/to/firefox-mcp-server"
    }
  }
}
```

## Best Practices

### When Testing Multiplayer Applications
1. Always use descriptive sessionId names ("host", "player1", "admin")
2. Specify sessionId explicitly in tool calls for clarity
3. Use `session_list` to verify active sessions
4. Take screenshots of each session for visual verification
5. Monitor debug feeds for real-time application behavior

### When Debugging Phoenix LiveView
1. Use `debug_websocket_messages` to track LiveView state updates
2. Monitor console logs for LiveView errors and warnings
3. Use `debug_network_activity` to track HTTP requests
4. Clear debug buffers periodically to manage memory

### Error Handling
- Always check if browser is launched before creating sessions
- Verify elements exist before interaction
- Use appropriate timeouts for `element_wait`
- Handle session closure gracefully in error scenarios

---
> Source: [JediLuke/firefox-mcp-server](https://github.com/JediLuke/firefox-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->

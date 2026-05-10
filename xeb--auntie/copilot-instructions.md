## auntie

> auntie is a Chrome browser automation CLI using the DevTools Protocol. It communicates with Chrome via WebSocket to control navigation, DOM interaction, screenshots, and more.

# auntie - Development Guide

## About

auntie is a Chrome browser automation CLI using the DevTools Protocol. It communicates with Chrome via WebSocket to control navigation, DOM interaction, screenshots, and more.

## Architecture

```
CLI (main.rs) → BrowserServer (browser.rs) → Chrome DevTools Protocol (WebSocket)
                     ↓
               BrowserDb (db.rs) → SQLite (~/.auntie/auntie.db)
```

### Key Components

1. **CLI** (`src/main.rs`)
   - Clap-based argument parsing
   - Subcommands for all browser operations
   - Delegates to BrowserServer for execution

2. **BrowserServer** (`src/browser.rs`)
   - WebSocket client for Chrome DevTools Protocol
   - Methods for navigation, DOM, screenshots, cookies
   - Synchronous blocking API (uses `recv_timeout`)

3. **Database** (`src/db.rs`)
   - SQLite via rusqlite
   - Tables: credentials, sessions, state
   - Stores login info and saved sessions

4. **Output** (`src/output.rs`)
   - TTY detection for human vs JSON output
   - Formatting for tabs, cookies, HTML, markdown

## File Structure

```
src/
├── main.rs      - CLI entry point, argument parsing, command dispatch
├── browser.rs   - BrowserServer implementation, DevTools Protocol client
├── db.rs        - SQLite database for credentials/sessions
└── output.rs    - Output formatting (human-readable and JSON)
```

## Chrome DevTools Protocol

The tool communicates with Chrome via WebSocket on the debugging port (default 9222).

### Connection Flow

1. Start Chrome with `--remote-debugging-port=9222`
2. Fetch `http://localhost:9222/json` to get WebSocket URL
3. Connect WebSocket to `webSocketDebuggerUrl`
4. Send JSON-RPC commands, receive responses

### Common CDP Methods Used

| Method | Purpose |
|--------|---------|
| `Page.navigate` | Navigate to URL |
| `Page.reload` | Reload page |
| `Runtime.evaluate` | Execute JavaScript |
| `Page.captureScreenshot` | Take screenshot |
| `Page.printToPDF` | Save as PDF |
| `DOM.getDocument` | Get DOM tree |
| `DOM.querySelector` | Find element |
| `Input.dispatchKeyEvent` | Keyboard input |
| `Input.dispatchMouseEvent` | Mouse clicks |
| `Network.getCookies` | Get cookies |
| `Network.setCookie` | Set cookie |

## Building

```bash
make build      # Debug build
make release    # Release build (optimized)
make install    # Install to ~/.cargo/bin
make test       # Run tests
make clean      # Clean artifacts
```

## Dependencies

- **clap**: CLI argument parsing
- **tungstenite**: WebSocket client
- **serde/serde_json**: JSON serialization
- **rusqlite**: SQLite database
- **anyhow**: Error handling
- **base64**: Screenshot encoding
- **dirs**: Home directory detection
- **atty**: TTY detection

## Design Decisions

1. **Synchronous API**: Uses blocking WebSocket for simplicity
2. **Detached Chrome**: `auntie start` detaches Chrome process so it survives CLI exit
3. **Port-based stop**: `auntie stop` finds Chrome via `lsof` on the debugging port
4. **SQLite storage**: Credentials and sessions persisted in user's home directory
5. **Auto JSON output**: Detects TTY to choose human vs JSON format

## Adding New Commands

1. Add variant to `Command` enum in `main.rs`
2. Add handler in `run()` function
3. Implement method in `BrowserServer` if needed
4. Update README.md with usage

## Error Handling

- Uses `anyhow::Result` for error propagation
- `BrowserResult` struct for operation outcomes (success, message, data)
- Exit code 1 on failures

## Testing

Currently no automated tests. Manual testing workflow:

```bash
auntie start
auntie open https://example.com
auntie page
auntie screenshot test.png
auntie stop
```

---
> Source: [xeb/auntie](https://github.com/xeb/auntie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->

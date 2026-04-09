## unifiedthreadtheory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unified Thread Theory is an Electron-based desktop application for managing email via FastMail's JMAP API. The app connects to FastMail, retrieves mailboxes and emails, and displays them in a clean interface.

## Development Commands

### Running the Application
```bash
npm start
```

### Installing Dependencies
```bash
npm install
```

### Building the Application
```bash
# Using electron-builder (available in devDependencies)
npx electron-builder
```

## Architecture

### Process Model

The application follows Electron's standard multi-process architecture:

- **Main Process** (`src/main/main.js`): Manages application lifecycle, creates windows, and handles IPC communication with the renderer
- **Renderer Process** (`src/renderer/`): Runs the UI (HTML/CSS/JS) in a Chromium environment
- **Preload Script** (`src/preload/preload.js`): Bridges main and renderer processes via `contextBridge`, exposing a limited API to the renderer

### Key Components

#### JMAP Client (`src/main/jmap-client.js`)
Custom implementation of FastMail JMAP client that wraps the `jmap-client` library:

- **NodeTransport class**: Custom HTTP transport layer for Node.js/Electron environment
  - Handles method name conversion between library's camelCase format (e.g., `getMailboxes`) and JMAP's Object/Verb format (e.g., `Mailbox/get`)
  - Converts response method names back to library's expected format (e.g., `Mailbox/get` → `mailboxes`)
  - Normalizes FastMail's mailboxIds format from object (`{"P3V": true}`) to array (`["P3V"]`)
  - Implements redirect following for both GET and POST requests

- **FastmailClient class**:
  - Authenticates using Bearer token (API token from FastMail)
  - Fetches session data from `https://api.fastmail.com/jmap/session`
  - Extracts `apiUrl` and `accountId` from session response
  - Provides methods: `connect()`, `getMailboxes()`, `getEmails(mailboxId, limit)`

#### IPC Communication
The main process exposes three IPC handlers:
- `connect-fastmail`: Authenticates with FastMail using username and API token
- `get-mailboxes`: Retrieves all mailboxes for the authenticated account
- `get-emails`: Retrieves emails for a specific mailbox

The preload script exposes these as:
- `window.api.connectFastmail({username, password})`
- `window.api.getMailboxes()`
- `window.api.getEmails(mailboxId)`

### JMAP Implementation Details

The JMAP client implements custom request/response formatting:

**Request Format**: The library sends method calls as arrays, but JMAP requires wrapping in a request object with `using` (capabilities) and `methodCalls` fields.

**Response Handling**: FastMail returns responses in JMAP format. The transport layer extracts `methodResponses` and converts method names for library compatibility.

**Authentication**: Uses Bearer token authentication. The client strips any existing "Bearer " prefix from the password field and applies it properly in headers.

### Security Configuration

The application uses Electron security best practices:
- `contextIsolation: true` - Isolates renderer from Node.js environment
- `nodeIntegration: false` - Prevents direct Node.js access in renderer
- All main process APIs exposed through preload script's `contextBridge`

## Dependencies

- **electron**: Desktop application framework
- **jmap-client**: Base JMAP protocol library (wrapped with custom transport)
- **sqlite3**: Database support (installed but not currently used)
- **electron-builder**: Build tooling for creating distributable packages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/climbingblake)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/climbingblake)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

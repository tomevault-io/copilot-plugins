## open-computer-use

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Coasty** is a full-stack AI collaboration platform with computer automation capabilities. It features a Next.js frontend with a FastAPI Python backend that orchestrates multi-agent AI systems capable of browser automation, terminal operations, and desktop control through containerized virtual machines. A cross-platform Electron desktop app provides a lightweight overlay that executes AI agent commands directly on the user's local machine.

## Architecture

### Frontend (Next.js 15 + React 19)
- **Framework**: Next.js 15 with App Router, TypeScript, Tailwind CSS
- **State Management**: Zustand stores for chat, models, user, and sessions
- **Key Libraries**:
  - Vercel AI SDK (`ai`) for streaming LLM responses
  - Radix UI for accessible components
  - Supabase for authentication and database
  - Stripe for billing/subscriptions
- **Provider System**: Multi-provider AI support (OpenAI, Anthropic, Azure, Google, Mistral, xAI, OpenRouter, Perplexity)

### Backend (Python FastAPI)
- **Framework**: FastAPI with async/await patterns
- **Key Services**:
  - `multi_agent_executor.py`: Orchestrates multi-agent task execution with browser, terminal, and desktop agents
  - `vm_control.py`: WebSocket-based VM control with persistent connections and auto-reconnection
  - `database.py`: Supabase integration for user data, chats, and billing
  - `agent_billing.py`: Tracks usage and credits for agent sessions
  - `search.py`: Google Custom Search API integration
- **API Routes**: `/api/chat`, `/api/models`, `/api/search`, `/api/vm`, `/api/billing`, `/api/files`

### VM Agent System
- **Architecture**: Docker containers running Ubuntu 22.04 with XFCE desktop
- **Agent Types**:
  - **Browser Agent**: Web automation using Chrome with remote debugging (search-first strategy)
  - **Terminal Agent**: Command execution and file operations
  - **Desktop Agent**: UI automation with screenshot analysis
- **Communication**: WebSocket protocol on port 8080 (8081 for localhost)
- **Tools**: Each agent has specialized tools (browser navigation, terminal commands, desktop controls)

### Electron Desktop App (`electron/`)

A cross-platform Electron app (v40.6.0) that runs as a floating overlay on the user's desktop, executing AI agent commands locally instead of in a remote VM.

- **Build System**: electron-vite + electron-builder, React 19 + Tailwind CSS renderer
- **Version**: 1.5.0 (`com.coasty.desktop`)
- **Key Dependencies**: `puppeteer-core` (browser automation), `ws` (WebSocket client)

#### Main Process (`electron/src/main/`)

- **`index.ts`**: App entry — creates frameless transparent window, system tray, registers IPC handlers, auto-updater
- **`auth.ts`**: Google OAuth via Supabase implicit flow — spins up local HTTP server on random port for callback, extracts tokens from URL fragment via HTML redirect trick, auto-refreshes tokens 5min before expiry
- **`ws-bridge.ts`**: Persistent WebSocket to backend `/api/electron/ws` — sends system info as URL params, auth credentials in first message body (not URL), auto-reconnect with exponential backoff (max 15s), 30s heartbeat
- **`window-manager.ts`**: Three modes — `auth` (400x500 centered), `compact` (360x56 top-center pill), `expanded` (400x520 chat panel). Smooth animation (320ms quintic ease-out), always-on-top management, opacity control (0.15–1.0), hides before screenshots
- **`local-executor.ts`**: Command handler registry (50+ commands) — maps backend command names to local handlers, normalizes params (filepath→path, find→old_text), auto-hides overlay during UI interactions
- **`desktop-automation.ts`**: Platform-specific mouse/keyboard/scroll/drag operations
- **`browser-automation.ts`**: Puppeteer-core controlling installed Chrome/Edge/Brave with isolated temp user-data-dir
- **`terminal.ts`**: Session-based shell execution (PowerShell on Windows, bash on Unix), 30s timeout
- **`file-ops.ts`**: File system CRUD (read, write, edit, append, delete, directory listing)
- **`screenshot.ts`**: Electron `desktopCapturer` API, resized to max 1280px, JPEG 70% quality
- **`permissions.ts`**: macOS-only — checks Screen Recording and Accessibility permissions
- **`auto-updater.ts`**: Generic update provider at `https://updates.coasty.ai`, checks every 4 hours

#### Platform-Specific Implementations

**Windows:**

- **Desktop Automation**: PowerShell + user32.dll P/Invoke (`mouse_event`, `keybd_event`, `SendKeys`)
  - Click/double-click via `System.Windows.Forms.Cursor` + `mouse_event` DLL calls
  - Typing via `SendKeys::SendWait()` with special character escaping
  - Key combos via `keybd_event` with virtual key codes (supports Win key, modifiers)
  - Scroll via `MOUSEEVENTF_WHEEL` (120 units per notch)
  - Drag via cursor position + mousedown/mouseup sequence
- **Browser Discovery**: Checks Program Files, Program Files (x86), LocalAppData for Chrome/Edge/Brave; falls back to `where.exe` PATH search
- **Terminal**: Uses `powershell.exe -Command` for all shell execution
- **Window Management**: PowerShell + `user32.dll ShowWindow` for minimize/maximize/restore/close; `Microsoft.VisualBasic.Interaction.AppActivate` for window switching
- **Window Z-Order Workaround**: Transparent frameless windows lose always-on-top on Windows; fix is hide→apply bounds→show sequence on auth→overlay transition, with retries at 600ms/1200ms/2000ms/3000ms
- **Build**: NSIS installer, allows custom install directory, desktop + start menu shortcuts

**macOS:**

- **Desktop Automation**: Swift scripts via CoreGraphics + osascript
  - Click/double-click via `CGEvent` with proper `mouseEventClickState` for double-clicks
  - Typing via `osascript 'tell application "System Events" to keystroke'`
  - Key combos via osascript with modifier mapping (ctrl→`control down`, cmd→`command down`) + CGKeyCode for special keys
  - Scroll via `CGEvent(scrollWheelEvent2Source:)` with line units
  - Drag via CGEvent sequence (leftMouseDown → leftMouseDragged → leftMouseUp)
- **Browser Discovery**: Checks `/Applications/` for Google Chrome, Microsoft Edge, Brave Browser `.app` bundles; falls back to `which`
- **Terminal**: Uses `/bin/bash -c` for shell execution
- **Permissions**: Checks Screen Recording (`getMediaAccessStatus` + actual capture fallback) and Accessibility (`isTrustedAccessibilityClient`); provides System Preferences deep links
- **Window Behavior**: `setVisibleOnAllWorkspaces` for macOS Spaces, dock icon set separately via `app.dock.setIcon`
- **App Lifecycle**: Does not quit on window close (`window-all-closed` ignored on darwin), `activate` event re-creates window
- **Build**: DMG + ZIP targets, hardened runtime, code signing, notarization, entitlements for accessibility/screen recording

**Linux:**

- **Desktop Automation**: `xdotool` for mouse/keyboard, `wmctrl` for window management
- **Browser Discovery**: Checks `/usr/bin/` for google-chrome, chromium-browser, microsoft-edge, brave-browser
- **Build**: AppImage target

#### Renderer Process (`electron/src/renderer/`)

- **Zustand Stores**: `auth-store` (user session), `connection-store` (WebSocket state), `chat-store` (messages, tool invocations, chat CRUD), `window-store` (mode sync)
- **SSE Parser** (`lib/sse-parser.ts`): Parses backend events — text (0), error (3), tool call (9), tool result (a), reasoning (g), finish (d)
- **Key Components**: `AuthScreen` (Google OAuth), `Overlay` (pill bar + expanded chat panel), `PermissionsGuard` (macOS permission prompts), `MessageList`, `ChatHistory`

#### Preload Bridge (`electron/src/preload/`)

Exposes `window.coasty` TypeScript API: auth methods, bridge control, chat CRUD, credits, window control (mode, opacity), update control, macOS permissions, event listeners

#### IPC Communication

- **Auth**: `auth:sign-in`, `auth:sign-out`, `auth:get-session`, `auth:get-token`
- **WebSocket Bridge**: `bridge:connect`, `bridge:disconnect`, `bridge:get-state`
- **Chat CRUD**: Create, list, get messages, update, delete — all call FastAPI backend with Bearer token
- **Window**: `window:set-mode`, `window:set-opacity`, `window:get-opacity`
- **Permissions**: `permissions:check`, `permissions:request-accessibility`, `permissions:open-screen-recording`
- **Updates**: `update:get-status`, `update:get-version`, `update:install`
- **Machine ID**: Deterministic UUID v5 hash of `electron-{user_id}-{hostname}-{username}-{platform}`

#### Backend Integration

- `/api/electron/ws` — WebSocket for receiving and executing commands
- `/api/chats/create`, `/api/chats/list`, `/api/chats/{id}/messages` — Chat persistence
- `/api/chat/` — SSE streaming chat responses
- `/api/billing/credits/balance` — Credit checks

### Key Design Patterns

#### Multi-Agent Execution Flow
1. **Task Planning**: LLM decomposes user request into sequential subtasks
2. **Agent Assignment**: Each subtask assigned to specialized agent (browser/terminal/desktop)
3. **Sequential Execution**: Tasks execute in order (no dependencies system)
4. **Context Passing**: Previous task summaries passed to next task for context
5. **Streaming**: All execution streams via Server-Sent Events to frontend

#### Provider Architecture
- Located in `lib/providers/` and `backend/app/providers/`
- Each provider implements streaming chat with tool calling
- Frontend providers handle model selection and API routing
- Backend providers execute tools and manage agent workflows

#### State Management
- **Chat Store** (`lib/chat-store/`): Manages conversations, messages, attachments
- **Model Store** (`lib/model-store/`): Available models and provider configurations
- **User Store** (`lib/user-store/`): User profile and authentication state
- **VM Store** (`lib/vm-store/`): Virtual machine sessions and connections

## Development Commands

### Frontend Development

```bash
# Install dependencies
npm install

# Development server (with Turbopack)
npm run dev

# Production build
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Linting
npm run lint
```

### Backend Development

```bash
# Navigate to backend directory
cd backend

# Create virtual environment (first time)
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run development server (from backend directory)
python main.py

# Or use the helper script
# Windows:
.\run_backend.bat
# Linux/Mac:
./run_backend.sh
```

### Electron Desktop App Development

```bash
# Navigate to electron directory
cd electron

# Install dependencies
npm install

# Development mode (with hot reload)
npm run dev

# Build (compile TypeScript)
npm run build

# Package for current platform
npm run package

# Package for specific platform
npm run package:win    # Windows NSIS installer
npm run package:mac    # macOS DMG + ZIP
npm run package:linux  # Linux AppImage
```

### Docker Deployment

```bash
# Build and start all services
docker-compose up --build

# Start services in detached mode
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# AI Desktop container (separate compose file)
docker-compose -f docker-compose.ai-desktop.yml up --build
```

### Testing

```bash
# Backend tests
cd backend
pytest

# Run specific test file
pytest tests/test_specific.py

# Run with coverage
pytest --cov=app tests/
```

## Environment Configuration

### Frontend Environment Variables (.env)
- `NEXT_PUBLIC_SUPABASE_URL`: Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase anonymous key
- `SUPABASE_SERVICE_ROLE`: Supabase service role key (server-side)
- `CSRF_SECRET`: CSRF protection secret (required)
- `ENCRYPTION_KEY`: For encrypting user API keys (required for BYOK)
- `PYTHON_BACKEND_URL`: Backend API URL (default: http://0.0.0.0:8001)
- `NEXT_PUBLIC_BACKEND_URL`: Public backend URL (default: http://localhost:8001)
- Azure credentials for VM provisioning (AZURE_*)
- Stripe keys for billing (STRIPE_*)
- Google Search API keys (GOOGLE_SEARCH_*)

### Backend Environment Variables (backend/.env)
- `DEBUG`: Enable debug mode (true/false)
- `CORS_ORIGINS`: Allowed CORS origins (comma-separated)
- `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE`: Supabase config
- `CSRF_SECRET`, `ENCRYPTION_KEY`: Security keys (must match frontend)
- `GOOGLE_SEARCH_KEY`, `GOOGLE_SEARCH_CX`: Google Custom Search API

### Electron Environment Variables (electron/.env)

- `COASTY_BACKEND_URL`: Backend API endpoint (default: `http://localhost:8001`)
- `NEXT_PUBLIC_SUPABASE_URL`: Supabase auth/DB URL (shared with frontend)
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase anon key (shared with frontend)
- These are injected at build time via electron-vite `define` config

See `.env.example` and `backend/.env.example` for complete configuration templates.

## Code Organization

### Frontend Structure
- `app/`: Next.js app directory with routes and layouts
  - `c/[chatId]/`: Individual chat pages
  - `api/`: API route handlers (Next.js API routes)
  - `auth/`, `billing/`, `account/`: Feature-specific pages
- `components/`: Reusable React components
  - `ui/`: shadcn/ui components (Radix UI based)
  - `common/`: Shared components (chat interface, message display)
  - `prompt-kit/`: Prompt-related components
- `lib/`: Business logic and utilities
  - `providers/`: AI provider implementations
  - `chat-store/`, `model-store/`, `user-store/`: Zustand state stores
  - `supabase/`: Database client and queries
  - `services/`: Service layer (API calls, utilities)

### Backend Structure
- `backend/app/`
  - `api/routes/`: FastAPI route handlers
  - `services/`: Core business logic
    - `multi_agent_executor.py`: Multi-agent orchestration
    - `vm_control.py`: VM WebSocket management
    - `database.py`: Supabase operations
    - `agent_billing.py`: Usage tracking
  - `core/`: Configuration, middleware, logging
  - `models/`: Pydantic data models
  - `providers/`: AI provider integrations
  - `utils/`: Utility functions

### Electron Structure

- `electron/`
  - `src/main/`: Main process — app lifecycle, IPC, WebSocket bridge, automation modules
    - `index.ts`: App entry, window creation, tray, IPC registration
    - `auth.ts`: Supabase Google OAuth with local HTTP callback server
    - `ws-bridge.ts`: WebSocket client to backend with auto-reconnect
    - `window-manager.ts`: Window modes, animation, opacity, screenshot hiding
    - `local-executor.ts`: Command dispatch registry (50+ commands)
    - `desktop-automation.ts`: Platform-specific mouse/keyboard (Win32/macOS/Linux)
    - `browser-automation.ts`: Puppeteer-core browser control
    - `terminal.ts`: Shell execution (PowerShell/bash)
    - `file-ops.ts`: File system operations
    - `screenshot.ts`: Desktop capture via Electron API
    - `permissions.ts`: macOS permission checks
    - `auto-updater.ts`: Auto-update lifecycle
  - `src/preload/`: Context bridge exposing `window.coasty` API
  - `src/renderer/`: React 19 UI
    - `stores/`: Zustand stores (auth, connection, chat, window)
    - `components/`: AuthScreen, Overlay, MessageList, PermissionsGuard
    - `hooks/`: `useChatSubmit` for chat message flow
    - `lib/`: API client, SSE parser, utilities
  - `build/`: Icons (ico/icns/png), macOS entitlements plist
  - `electron-builder.yml`: Build config (NSIS, DMG, AppImage)
  - `electron.vite.config.ts`: Vite config with env injection

### Docker Structure

- `docker/ai-desktop/`: Ubuntu desktop container with AI agents
  - Includes Chrome, Node.js, Python, automation tools
  - WebSocket server for agent communication
  - VNC server for remote desktop access

## Key Workflows

### Adding a New AI Provider

1. **Frontend**: Create provider in `lib/providers/your-provider.ts`
   - Implement `streamChat()` method with tool calling support
   - Add to `lib/providers/index.ts`

2. **Backend**: Add provider support in `backend/app/providers/`
   - Configure API keys in environment
   - Update model lists in `models.py`

### Creating a New Agent Type

1. Add agent type to `AgentType` enum in `multi_agent_executor.py`
2. Create agent prompt in `_get_*_agent_prompt()` method
3. Define agent tools in `_get_*_tools()` method
4. Update task planner to recognize new agent type

### Adding New VM Tools

1. Create tool function in `backend/app/api/routes/chat_vm_tools.py`
2. Define tool schema (name, description, parameters)
3. Add tool to appropriate agent's tool list in `multi_agent_executor.py`
4. Implement tool execution in VM agent server (if needed)

### Adding a New Electron Local Command

1. Create handler function in the appropriate module (`desktop-automation.ts`, `browser-automation.ts`, `terminal.ts`, `file-ops.ts`, or a new module)
2. Register the command name → handler mapping in `local-executor.ts` `registerHandlers()`
3. If the command involves UI interaction (clicks, typing), wrap with `this.withOverlayHidden()`
4. Add parameter normalization in `normalizeParams()` if backend sends different param names
5. Backend sends commands via the WebSocket bridge as `{ type: 'command', data: { command, parameters } }`

### Adding Platform-Specific Desktop Automation

1. Add the function in `desktop-automation.ts` with `process.platform` branching
2. **Windows**: Use `runPowershell()` with user32.dll P/Invoke via `Add-Type @"..."@`
3. **macOS**: Use `runSwift()` for CoreGraphics or `runBash()` with `osascript` for System Events
4. **Linux**: Use `runBash()` with `xdotool` or `wmctrl`
5. Register in `local-executor.ts` and wrap with `withOverlayHidden()` if it interacts with the desktop

## Important Technical Details

### WebSocket Connection Management
- VM connections are persistent with auto-reconnection
- Heartbeat mechanism prevents stale connections
- Connection reuse minimizes latency
- Password authentication for VNC access

### Tool Response Handling
- Tool responses are truncated to prevent context overflow (5000 chars)
- `frontendScreenshot` field is preserved and not sent to model
- Screenshots are compressed (JPEG, 1280x720 max) before transmission

### Streaming Architecture
- All AI responses stream via Server-Sent Events (SSE)
- Tool calls and results stream separately from text
- Frontend accumulates chunks and updates UI reactively
- `finish` event signals completion with full content

### Task Execution Rules
- Tasks execute sequentially (no parallel execution)
- Each task receives context from all previous completed tasks
- Tasks can request user input via `[NEED_USER_INPUT]` markers
- Execution stops if agent encounters critical blocker

### Browser Agent Strategy
- **Search-First**: Always use Google Search before opening browser
- **Minimal Browsing**: Only open browser when action is required (forms, clicks, purchases)
- **State Validation**: Use `browser_state()` to verify actions
- **Tab Management**: Reuse tabs instead of excessive navigation

### Security Considerations

- CSRF protection on all state-changing operations
- API keys encrypted with `ENCRYPTION_KEY` (BYOK feature)
- Rate limiting on backend endpoints
- Supabase Row Level Security (RLS) for data access
- No credentials stored in VM environments
- **Electron**: Context isolation enabled, node integration disabled, sandbox=false (required for native modules)
- **Electron**: Auth tokens sent in WebSocket message body, not URL params (avoids proxy/CDN logging)
- **Electron**: Window title passed via env var (`_COASTY_WIN_TITLE`) to avoid shell injection in window switching

## Common Development Tasks

### Adding a New Feature
1. Design API endpoints in `backend/app/api/routes/`
2. Implement business logic in `backend/app/services/`
3. Create frontend components in `components/`
4. Add state management in appropriate store (`lib/*-store/`)
5. Wire up API calls in `lib/services/` or route handlers

### Debugging Electron Desktop App Issues

1. Check WebSocket bridge connection in console — `[WS Bridge]` log prefix shows connect/disconnect/auth events
2. Verify backend `/api/electron/ws` endpoint is running and accepting connections
3. On Windows: if overlay loses always-on-top, check `window-manager.ts` z-order workaround logic
4. On macOS: if automation fails, check Screen Recording + Accessibility permissions via `permissions.ts`
5. Browser automation: ensure Chrome/Edge/Brave is installed; Puppeteer uses isolated temp profile to avoid locks
6. If clicks/typing don't work: verify overlay is hiding before desktop actions (`withOverlayHidden` wrapper)
7. Auth issues: check local HTTP callback server port binding, Supabase OAuth redirect URL config

### Debugging VM Agent Issues
1. Check WebSocket connection status in `vm_control.py` logs
2. Verify agent tools are registered in `multi_agent_executor.py`
3. Test tool execution with reduced context
4. Check container logs: `docker logs <container-id>`
5. Verify VNC connection: `ws://localhost:8081` (localhost) or `ws://<ip>:8080`

### Optimizing Performance
1. **Frontend**: Use React.memo for expensive components, lazy load routes
2. **Backend**: Enable caching in `cache.py`, optimize database queries
3. **Streaming**: Batch small chunks, compress screenshots
4. **VM**: Reuse connections, minimize tool calls, truncate responses

### Database Migrations
- Supabase migrations handled via Supabase Dashboard or CLI
- Schema changes require updating Supabase types in `types/supabase.ts`
- Run `supabase gen types typescript` to regenerate types

## Ports and Services

- **Frontend**: 3000 (Next.js dev server)
- **Backend**: 8001 (FastAPI server)
- **VM Agent WebSocket**: 8080 (remote), 8081 (localhost)
- **VNC**: 5900 (desktop access)
- **Supabase**: Hosted service (URLs in .env)

## Additional Notes

- **Frontend uses React Server Components** where applicable for better performance
- **Backend runs on uvicorn** with auto-reload in development
- **VM containers** are ephemeral and should be treated as stateless
- **Billing system** tracks agent usage by session duration
- **Multi-model support** allows users to switch providers mid-conversation
- **Screenshot compression** is critical for performance (JPEG, 70% quality)
- **Electron app** launches at login (packaged builds), runs as always-on-top overlay on all virtual desktops
- **Electron auto-updates** via generic provider at `https://updates.coasty.ai` (checks every 4 hours)
- **Electron browser automation** uses `puppeteer-core` with temp user-data-dir to avoid Chrome profile locks
- **Electron overlay** auto-hides during desktop automation to prevent interfering with clicks/screenshots

---
> Source: [coasty-ai/open-computer-use](https://github.com/coasty-ai/open-computer-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

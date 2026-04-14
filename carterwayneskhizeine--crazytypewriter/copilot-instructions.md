## crazytypewriter

> **Startup:** `docker compose up -d --build` (do NOT use npm)

# Power Mode Typer - Developer Guide

**Startup:** `docker compose up -d --build` (do NOT use npm)

## Project Overview
Gamified text editor with particle effects, screen shake, and combo tracking. Built with React 19, TypeScript, Tailwind CSS.

## Core Features
- Particle System (canvas-based, spawns at caret position)
- Screen Shake (intensity scales with combo count)
- Combo System with Dynamic HUD (follows cursor position)
- 3 Themes: Retro Terminal, VS Code Modern, Modern Wild White
- Markdown Preview, Copy to Clipboard, Send to Server
- **User Authentication**: Login/logout with NextAuth 5 credentials provider
- **Multi-Device Sync**: Real-time sync via WebSocket (requires login)
- Responsive Design

## Tech Stack
- React 19 + TypeScript
- Tailwind CSS (CDN) + Lucide React Icons
- HTML5 Canvas for particles (`requestAnimationFrame`)
- CSS Keyframes for UI shake
- Deployment: Docker + Nginx (multi-stage build)

## Core Architecture

### State Management (App.tsx)
- Uses `useState` + `useRef` for performance
- **Critical:** Particles stored in `particlesRef.current` NOT `useState` to avoid render cycle overhead
- Key state: `text`, `combo`, `config`, `caretY`, `hudSide`
- **Authentication state**:
  - `user: User | null` - Current logged-in user (null if not authenticated)
  - `showLoginModal: boolean` - Controls login modal visibility
  - `loginForm: { email, password }` - Login form input values
  - `loginError: string` - Error message to display in modal
  - `isLoggingIn: boolean` - Loading state during login process

### Authentication System
**NextAuth 5 Credentials Provider** with localStorage session persistence:

**Login Flow:**
1. User clicks Lock icon → modal opens
2. Fetch CSRF token from `/api/auth/csrf`
3. Submit email/password to `/api/auth/callback/credentials` with `redirect: 'manual'`
4. Request includes: `{csrfToken, email, password}` (form-encoded)
5. Check for `opaqueredirect` response type or 302/301 status (indicates success)
6. On success, fetch user session from `/api/auth/session`
7. Backend sets HttpOnly session cookie
8. User info stored in `localStorage` for session persistence
9. Modal closes, Send button becomes visible

**No Page Redirect:**
- Login completes without page refresh using `redirect: 'manual'`
- Prevents NextAuth from redirecting to wrong URL
- Session fetch happens client-side after successful login

**Session Management:**
- On app mount: check `localStorage` for stored user data
- On logout: clear `localStorage` and call `/api/auth/signout` with callbackUrl
- Send API requires authentication (401 triggers re-login prompt)
- All authenticated requests use `credentials: 'include'` to send cookies

**User Data Structure:**
```typescript
interface User {
  id: string;        // User ID (string for NextAuth)
  email: string;     // User email (used for login)
  name?: string;     // Optional display name
}
```

**UI Components:**
- **Login Button**: Lock icon (logged out) → Display name or email + X icon (logged in)
  - Shows `user.name` if available, otherwise `user.email`
- **Send Button**: Only visible when `user` state exists
- **Undo Button**: Only visible when `user` state exists, requires authentication
- **Sync Status**: Cloud icon with status dots (only visible when logged in)
  - Green: Connected, Red: Disconnected, Yellow (pulsing): Pending changes
- **Login Modal**: Email input (not username), theme-specific styling, backdrop blur, Enter key support
  - Terminal theme: `[ LOGIN ]` header, green borders
  - VS Code theme: Clean dark modal with blue accents
  - Modern theme: White modal with gold borders

### Caret Tracking (utils/caret.ts)
Uses **Mirror Div Strategy** to get exact cursor position:
1. Creates hidden `<div>` copying textarea's computed styles
2. Inserts text up to cursor (`selectionStart`)
3. Appends `<span>` for cursor
4. Calculates span coordinates relative to viewport

### Dynamic HUD
- Vertical: Follows `caretY`
- Horizontal: Flips side based on cursor X position (left 50% → shows right, right 50% → shows left)

## Theme System
Three themes in cyclic rotation: Terminal → VS Code → Modern → Terminal
- **Terminal:** Green monochrome, CRT effects, block cursor
- **VS Code:** Dark theme, clean flat design, thin cursor
- **Modern:** White background, neon pink/cyan/yellow accents, random particle colors

**Implementation:**
- `themeMode` state: `'terminal' | 'vscode' | 'modern'`
- Theme-specific color arrays: `TERMINAL_LEVEL_COLORS`, `VSCODE_LEVEL_COLORS`, `MODERN_LEVEL_COLORS`
- Helper function: `getLevelColors(theme, level)`
- Theme flags: `isTerminal`, `isVSCode`, `isModern`
- Modern theme uses `MODERN_NEON_COLORS` for random particles regardless of power level

## Power Levels (Combo Thresholds)
| Level | Threshold | Effects |
|-------|-----------|---------|
| None | 0-9 | Minimal effects |
| Power | 10-29 | Light screen shake |
| Super Power | 30-59 | Moderate shake |
| Many Power | 60+ | Intense shake + "use with caution" warning |

Colors differ per theme (green/blue/neon), Modern theme has expanded neon palette at higher levels.

## Key Files
- `App.tsx` - Main component, state management, particle physics, NextAuth authentication logic
- `types.ts` - TypeScript interfaces (PowerConfig, Particle, User, LoginResponse, SyncStatus)
- `utils/caret.ts` - Cursor position tracking
- `nginx/default.conf` - SPA routing + API proxy (`/api/` → external server via `POST_HOST` env var)
- `docker-compose.yml` - Port mapping `5111:80`
- `index.html` - Theme-specific login modal CSS styles (login-modal-*, login-input-*, login-btn-*)

## Important Development Notes

### Performance
- Particles MUST use `particlesRef.current` - never `useState` (causes input lag)
- `utils/caret.ts` DOM read/write is expensive - keep efficient
- Avoid heavy computations in `handleInput` (high-frequency keydown events)

### Adding New Themes
1. Update `ThemeMode` type
2. Create color constant array (e.g., `NEWTHEME_LEVEL_COLORS`, `NEWTHEME_NEON_COLORS`)
3. Update `getLevelColors()` and all color helper functions
4. Add theme flag and update all conditional rendering
5. Update theme toggle button logic
6. Add CSS styles in `index.html` (cursor, buttons, scrollbar, markdown preview)
7. **Add login modal CSS classes** for the new theme:
   - `.login-modal-{themename}` - Main modal container
   - `.login-header-{themename}` - Modal header
   - `.login-input-{themename}` - Email/password input fields
   - `.login-btn-{themename}` - Login button
   - `.login-error-{themename}` - Error message display

### Adding New Levels
1. Update `PowerLevel` enum in `types.ts`
2. Update `LEVEL_COLORS` and `COMBO_THRESHOLDS` in `App.tsx`
3. Add shake keyframe in `index.html` if needed

### Layout Changes
- Main container uses `relative`, HUD uses `absolute`
- Watch `scrollTop` and `window.scrollY` interactions when modifying layout

### Authentication Security
**DO:**
- Use `credentials: 'include'` for all authenticated requests (enables cookie handling)
- Use `redirect: 'manual'` in fetch to prevent NextAuth from redirecting to wrong URL
- Check `response.type === 'opaqueredirect'` or status 302/301 to detect successful login
- Store only non-sensitive user info in `localStorage` (`{id, email, name}`)
- Use `type="email"` for email inputs and `type="password"` for password inputs
- Let browser handle HttpOnly cookies automatically
- Always fetch CSRF token before submitting login credentials

**DON'T:**
- Store passwords or session tokens in `localStorage` (security risk)
- Manually extract or set cookies in JavaScript
- Send credentials in URL parameters
- Log passwords to console
- Follow NextAuth's redirects automatically (causes navigation to wrong URL)

**Error Handling:**
- Invalid credentials: Show error message in modal (don't close modal)
- Network errors: Show "Network error. Please try again." message
- Session expired (401): Clear localStorage, show login modal with error
- CSRF token fetch failure: Show "Network error" message
- Session fetch after login: Show error but keep user in logging-in state

**API Endpoints:**
All proxied through nginx to `https://whitenote.goldie-rill.top`:

- `GET /api/auth/csrf` - Get CSRF token for login
- `POST /api/auth/callback/credentials` - Submit login credentials
- `GET /api/auth/session` - Get current user session
- `POST /api/auth/signout` - Logout user
- `POST /api/messages` - Send message (requires authentication)
- `POST /api/sync/documents/undo` - Undo to previous document version (requires authentication)

## Multi-Device Real-Time Sync

**Architecture:** WebSocket-based real-time synchronization (similar to Google Docs)

### How It Works
When 2+ devices login with the same account:
1. **Auto-sync**: Content syncs automatically after typing stops (configurable debounce, default 5s)
2. **Real-time updates**: WebSocket pushes changes instantly to other devices
3. **Persistence**: Content saved to SQLite database, survives browser closure
4. **Conflict resolution**: Optimistic locking with version numbers (Last-Write-Wins)

### Tech Stack
- **Backend**: Node.js + Express + Socket.io + SQLite (sql.js)
- **Frontend**: socket.io-client + custom `useSync` hook
- **Deployment**: Docker container at `sync-server:3001`

### Key Files
- `sync-server/` - Backend sync service
  - `src/config/database.ts` - SQLite database management
  - `src/models/UserDocument.ts` - Document model with version tracking
  - `src/socket/index.ts` - WebSocket event handlers
  - `src/services/syncService.ts` - Sync logic and conflict resolution
  - `src/routes/documents.ts` - REST API endpoints
- `hooks/useSync.ts` - React hook for WebSocket connection and sync logic
- `nginx/default.conf` - WebSocket proxy configuration (`/socket.io/`)

### Configuration
Environment variables (`.env`):
- `VITE_SYNC_DEBOUNCE_MS` - Debounce delay in ms (default: 5000)
  - Controls how long after typing stops before auto-sync
  - Lower values = faster sync but more server requests
  - Higher values = less frequent sync but better performance

### WebSocket Events
**Client → Server:**
- `content:update` - Send updated content with version
- `sync:request` - Request current document content

**Server → Client:**
- `content:loaded` - Initial document content on connection
- `content:updated` - Real-time update from another device
- `sync:success` - Update successful, includes new version
- `sync:conflict` - Version conflict detected
- `sync:error` - Sync operation failed

### Sync Status Indicators
Visual feedback in header (visible when logged in):
- **Green dot** - Connected to sync server
- **Red dot** - Disconnected from server
- **Yellow dot (pulsing)** - Pending changes waiting to sync
- **Cloud icon (pulsing)** - Currently syncing

### Important Implementation Notes
**Critical: Stable Function References**
- All callbacks passed to `useSync` MUST be wrapped in `useCallback`
- Prevents WebSocket reconnection on every render
- Example: `const handleContentReceived = useCallback((content) => setText(content), []);`

**Device Identification:**
- Each device gets unique ID stored in `localStorage` (`sync_device_id`)
- Prevents sync loops when receiving own updates
- Generated automatically on first visit

**Conflict Detection:**
- Client sends content with current version number
- Server checks version before updating
- Mismatch returns conflict error with server content
- Current implementation: Server wins (can be enhanced with user prompt)

**Performance:**
- Debouncing reduces server load significantly
- WebSocket connection persists for duration of session
- Updates only triggered on text changes, not every keystroke

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carterwayneskhizeine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carterwayneskhizeine)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

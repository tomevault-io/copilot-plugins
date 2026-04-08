## mini-meet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Mini Meet is a minimal 1:1 WebRTC video chat application. Users create a meeting via `/new`, get a unique URL (`/m/:id`), and share it with one peer. The app uses Express for HTTP/WebSocket signaling and browser WebRTC APIs for peer-to-peer media streaming.

## Development Commands

**Start development server:**
```bash
npm run dev
# Or: node --run dev
```
This runs `scripts/dev.js` which:
- Auto-detects local `.local` hostname via avahi (if available)
- Generates SSL certificate with mkcert for localhost + .local domain
- Starts server with HTTPS + DEBUG logging enabled
- Runs Tailwind CSS in watch mode via concurrently

**Individual processes (if needed):**
```bash
npm run watch-server  # Server only with --watch flag
npm run watch-build   # Tailwind CSS only
npm run build         # One-time Tailwind CSS build
```

**Production:**
```bash
npm start  # Runs src/server.js directly
```

**Logging:**
Enable specific namespaces via `DEBUG` env var:
```bash
DEBUG=mini-meet:*              # All logs
DEBUG=mini-meet:ws,mini-meet:beacon  # Only WebSocket + client beacons
DEBUG=mini-meet:ws:msg         # Verbose WebSocket messages
```

## Architecture

### Server Architecture (`src/server.js`)

**Single-file server with these components:**

1. **HTTP/HTTPS Server:** Express app that conditionally creates HTTP or HTTPS server based on `SSL_CERT_PATH` and `SSL_KEY_PATH` env vars

2. **Middleware Stack (order matters):**
   - Cache-Control headers (no caching)
   - `express.json()` body parser (must come early for `/log` endpoint)
   - Request logger with scoped debug loggers attached to `req.log`
   - Static file serving from `public/`
   - PostHog proxy (`/ph/static/*` MUST come before `/ph/*`)
   - Routes: `/new`, `/m/:id`, `/`, `/turn`, `/log`
   - Error handling + Rollbar

3. **WebSocket Signaling Server:**
   - Mounted at `/ws?roomId=xyz`
   - In-memory room registry: `Map<roomId, Set<WebSocket>>`
   - Enforces 1:1 limit (rejects 3rd connection with `room_full`)
   - First peer becomes initiator
   - Forwards signaling messages: `ready`, `offer`, `answer`, `candidate`, `bye`, `leave`

4. **Scoped Logger Pattern:**
   ```javascript
   req.log = {
     http: createLogger('mini-meet:http', { ip, roomId }),
     turn: createLogger('mini-meet:turn', { ip }),
     beacon: createLogger('mini-meet:beacon', { ip })
   };
   ```
   All logs auto-include `ip=X.X.X.X roomId=xyz` for easy filtering.

### Client Architecture (`public/meeting.js`)

**Single-page WebRTC client with these responsibilities:**

1. **Media Management:**
   - `startLocalMedia()`: Requests camera/mic, displays in all `[data-local-video]` elements
   - Track enable/disable for mute/video toggles
   - Camera switching on mobile (`swapCameraFacing()`)

2. **WebSocket Signaling:**
   - `connectWebSocket()`: Connects to `/ws?roomId=xyz`
   - Handles: `welcome`, `ready`, `offer`, `answer`, `candidate`, `bye`, `leave`, `room_full`
   - Auto-reconnects on disconnect (unless `isShuttingDown`)

3. **WebRTC Peer Connection:**
   - `setupPeerConnection(reason)`: Creates new RTCPeerConnection
   - Always tears down old connection before creating new one
   - Smart reconnection logic:
     - `welcome`: Only recreate if PC doesn't exist or is unhealthy
     - `bye`: Only tear down if PC is actually dead (preserves healthy connections)
     - `leave`: Always tear down (explicit peer exit)
   - Russian user detection: Skips Google STUN, filters TURN URLs to TCP-only

4. **UI State Management:**
   - Mobile: Detects orientation changes, switches between portrait/landscape layouts
   - Desktop: Toggle between side-by-side and overlay layouts (saved in localStorage)
   - Mobile overlay: Draggable + pinch-to-resize with touch events
   - Status indicator synced between portrait/landscape elements

5. **Client Beacons:**
   - `beacon(event, context)`: Sends to `/log` via `navigator.sendBeacon()`
   - Also forwards to PostHog (if configured) via `window.posthog.capture()`
   - Uses Blob with `application/json` type (plain objects send as `text/plain`)

### View Rendering (`src/views/`)

**Server-side HTML generation using `nanohtml`:**

- `layout.html.js`: Base HTML structure, injects Rollbar + PostHog snippets
- `meeting.html.js`: Meeting page with responsive mobile/desktop layouts
- `index.html.js`: Landing page
- `rollbar.html.js`, `posthog.html.js`: Script injection snippets

**Important:** Views use `html` tagged template literals from `nanohtml`. Server-side renders static HTML with embedded JS variables (e.g., `window.__IS_MOBILE__ = ${isMobile}`).

### PostHog Analytics Proxy

**Why:** Bypass ad blockers by making PostHog requests appear same-origin.

**Server proxy setup (`src/server.js`):**
```javascript
// Order matters! Static MUST come before general /ph proxy
app.use('/ph/static', createProxyMiddleware({
  target: 'https://eu-assets.i.posthog.com',
  changeOrigin: true,  // Sets Host header
  pathRewrite: { '^/': '/static/' }  // Express strips /ph/static, so rewrite / to /static/
}));

app.use('/ph', createProxyMiddleware({
  target: 'https://eu.i.posthog.com',
  changeOrigin: true,
  pathRewrite: { '^/ph': '' }
}));
```

**Client setup:**
- PostHog SDK loaded from CDN in `src/views/posthog.html.js`
- Initialized with `api_host: '/ph'` (routes all requests through proxy)
- Beacon function automatically sends events to both `/log` and PostHog

### TURN Server Integration

**Server (`/turn` endpoint):**
- Uses TURN REST API pattern (RFC 7635)
- Generates short-lived credentials: `username = timestamp:random_tag`, `credential = HMAC-SHA1(username, TURN_SECRET)`
- Returns `{ iceServers: [...], ttl }` for client

**Client:**
- Fetches `/turn` during peer connection setup
- Appends TURN servers to Google STUN (unless Russian user detected)
- Russian users: Skip Google STUN, filter TURN URLs to TCP-only (`transport=udp` stripped)

## Key Implementation Details

### Mobile Landscape Layout

Mobile devices switch between two layouts:
- **Portrait:** Header + footer with controls, overlay in bottom-right
- **Landscape:** Side status column + right controls column, no header/footer

Layout switching happens in `applyMobileLandscapeLayout()` triggered by:
- `resize` event
- `orientationchange` event
- `orientationQuery.matches` media query change

### Overlay Drag & Resize

**Mobile overlay:**
- Drag: Single touch point moves overlay
- Pinch: Two touch points resize via `overlayPinchState`
- Aspect ratio syncs with orientation changes
- Removes conflicting Tailwind classes (`!w-24`, `!w-40`) before setting inline width

**Desktop overlay:**
- Mouse drag only (no resize)
- Separate state management in `desktopOverlayDragState`

### Browser Compatibility Detection

`detectUnsupportedBrowser()` checks for:
1. Known in-app browsers (Instagram, Facebook, Telegram, etc.)
2. Missing `navigator.mediaDevices.getUserMedia`

If detected, shows modal with "Share Link" button (uses `navigator.share` if available).

### Page Exit Handling

Uses `pagehide` event (not `beforeunload`) for iOS Safari compatibility:
```javascript
window.addEventListener('pagehide', () => {
  markShuttingDown();
  try { send('bye'); } catch (_) {}
});
```

## Environment Variables

**Required for HTTPS (dev/production):**
- `SSL_CERT_PATH`, `SSL_KEY_PATH`: Paths to cert/key files

**Optional TURN:**
- `TURN_URLS`: Comma-separated TURN/TURNS URLs
- `TURN_SECRET`: Shared secret (matches coturn config)
- `TURN_TTL`: Credential lifetime in seconds (default 900)

**Optional Analytics:**
- `POSTHOG_API_KEY`: PostHog project API key (public, starts with `phc_`)
- `POSTHOG_API_HOST`: Defaults to `/ph` (proxy endpoint)
- `ROLLBAR_SERVER_ACCESS_TOKEN`, `ROLLBAR_CLIENT_ACCESS_TOKEN`
- `ROLLBAR_ENVIRONMENT`: e.g., `development`, `production`

**Server:**
- `PORT`: Server port (default 3003)
- `DEBUG`: Debug namespace filter (e.g., `mini-meet:*`)

## Common Patterns

### Adding New Server Logs

Use scoped loggers attached to `req` or create via `createLogger()`:
```javascript
req.log.http('some message');  // Auto-includes ip and roomId
const wsLogger = createLogger('mini-meet:ws', { ip, roomId });
wsLogger('connection established');
```

### Adding New Client Events

Add beacon call + PostHog will auto-receive it:
```javascript
beacon('event_name', { key: 'value' });
// Sends to /log + PostHog (if configured)
```

### Modifying WebRTC Reconnection Logic

Be careful when editing `welcome` and `bye` handlers. Current logic:
- Only recreate peer connection if it doesn't exist OR is unhealthy
- Preserves healthy connections during transient WebSocket drops
- `leave` message always tears down (explicit peer departure)

### Working with Views

Views use `nanohtml` tagged template literals. To add server-side variables:
```javascript
html`<script>window.__MY_VAR__ = ${myVar};</script>`
```

Access in client JS: `const myVar = window.__MY_VAR__;`

## Deployment

**Dokku (recommended):**
See README.md "Deploy on Dokku" section.

**Docker:**
Use provided `Dockerfile`. Set env vars via docker-compose or runtime flags.

**TURN server:**
Self-host with coturn (see `coturn/docker-compose.yml`) or use managed provider (Twilio, Cloudflare Calls).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artemave)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/artemave)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

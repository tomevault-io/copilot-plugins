## claude-remote

> The user is controlling you remotely from their phone via this app. They cannot restart the server manually. They cannot SSH in. They cannot fix things if you break them.

# Claude Remote

## CRITICAL: YOU ARE BEING OPERATED FROM A PHONE

The user is controlling you remotely from their phone via this app. They cannot restart the server manually. They cannot SSH in. They cannot fix things if you break them.

**If you fail to restart the server after making changes, the user loses all access to you.** This is not a minor inconvenience — it is a total loss of control. You will be unreachable until someone physically accesses the machine.

### Deployment Rules (NON-NEGOTIABLE)

- **ALWAYS use `make deploy`** to build + restart. This builds the client, restarts the systemd service, and verifies it came back up.
- **NEVER kill server processes directly** — you are running inside the server as a subprocess. Killing the server kills you.
- **NEVER skip the build step** — if the client doesn't build, the frontend will be broken and the user will see a blank page from their phone.
- **ALWAYS verify the build succeeds before restarting** — if `pnpm build` fails, DO NOT restart the server. Fix the build error first.
- If a port is stuck after a crash, use `make free-ports`.
- `make restart` for daemon-only restart (no rebuild).

### After ANY Code Change

1. Run `pnpm build` — if it fails, STOP and fix the error
2. Run `make deploy` — if it fails, STOP and investigate
3. The user will reconnect automatically once the server is back up

## Overview

Mobile chat interface for local Claude CLI with E2E encryption. Access Claude from phone via Cloudflare tunnel (or any HTTPS reverse proxy).

## Architecture

### Tech Stack

- **Vite + React** client app
- **Custom Node server** (`server.ts`) with WebSocket support
- **Tailwind CSS** for styling (CSS variables in `client/src/index.css`)
- **argon2** for PIN hashing
- **ws** for WebSocket server

### URL Mapping (Production)

- `CLIENT_URL` env var = web client (served static files)
- `SERVER_URL` env var = API server (WebSocket, REST)

### Directory Structure

```
├── server.ts              # HTTP + WebSocket server, port 6767
├── src/lib/
│   ├── crypto.ts          # ECDH P-256 + AES-256-GCM
│   ├── store.ts           # Config/device/conversation persistence
│   └── claude.ts          # Claude CLI spawning + stream parsing
├── client/src/
│   ├── pages/Chat.tsx     # Main chat interface
│   ├── pages/Home.tsx     # Pairing / status page
│   ├── components/        # UI components
│   └── lib/api.ts         # Authenticated fetch wrapper
├── Makefile               # deploy, restart, free-ports, docker-*
└── claude-remote.service  # systemd unit file
```

### Security Model

- E2E encryption: ECDH key exchange, AES-256-GCM per message
- PIN auth: argon2-hashed, rate limited (5 attempts/min/IP)
- All `/api/` routes require Bearer PIN (except limited `/api/status`)
- CORS restricted to known origins
- Path traversal protection on static files and projectId
- Timing-safe device lookup and PIN comparison

## Development Principles

### Error Handling: SEEK ERRORS, DON'T HIDE THEM

- **Always show errors to the user** — silent failures waste hours of debugging
- When implementing any flow (pairing, auth, network calls, crypto):
  1. Wrap each step in try-catch
  2. Log with context
  3. On failure, throw with context
  4. Surface errors in the UI immediately — red box, full error text, selectable
- **Never assume success** — if something can fail, show what happened
- **Verbose by default** — it's easier to remove logs than to add them when debugging

### UI: NO PLACEHOLDER/INOP ELEMENTS

- Never add UI elements (buttons, icons, etc.) that don't work yet
- No "TODO: implement" buttons — either implement it fully or don't add it
- Confusing inop UI is worse than no UI

## Key Libraries

### `src/lib/crypto.ts`

- `generateKeyPair()` — ECDH P-256
- `deriveSharedSecret(privateKey, peerPublicKey)` — ECDH derive
- `encrypt(plaintext, secret)` — AES-256-GCM, returns {iv, ct, tag}
- `decrypt(encrypted, secret)` — AES-256-GCM
- Key serialization helpers (base64 <-> Buffer)

### `src/lib/store.ts`

- Config dir: `~/.config/claude-remote/`
- `loadDevice()` / `saveDevice()` — device.json
- `loadConfig()` / `saveConfig()` — config.json (PIN hash)
- `hashPin(pin)` / `verifyPin(pin, hash)` — argon2

### `src/lib/claude.ts`

- `spawnClaude(message, onEvent, signal)` — spawn CLI with streaming
- Parse JSON stream events: content_block_start, content_block_delta
- Map to simplified events: {type: 'thinking'|'text', text, done}
- Handle cancel via AbortSignal

## Pairing Flow

1. Server generates ECDH keypair + random token on startup (if no device)
2. Desktop shows QR code with URL: `{baseUrl}/pair/{token}`
3. Phone scans, opens URL, generates own ECDH keypair
4. Phone POSTs its public key to `/pair/{token}`
5. Server derives shared secret, stores device, returns its public key
6. Phone derives shared secret, stores locally
7. Server now shows "paired" status, phone redirects to chat

## Message Flow

1. Phone connects WebSocket, sends encrypted `{type: 'auth', pin}`
2. Server decrypts, verifies PIN hash, sends encrypted `{type: 'auth_ok'}`
3. Phone sends encrypted `{type: 'message', text}`
4. Server spawns `claude --print --output-format stream-json`
5. Server streams encrypted `{type: 'thinking'|'text', text, done}` events
6. Phone decrypts and displays in real-time

## Dev Logging

All dev commands tee output to log files in `logs/`:

- `pnpm dev` → `logs/server.log` + `logs/client.log`
- `pnpm dev:server` → `logs/server.log`
- `pnpm dev:client` → `logs/client.log`
- `pnpm start` → `logs/server.log`

Tail logs with:

- `pnpm logs:server`
- `pnpm logs:client`

The `logs/` directory is gitignored.

## Verification

1. `make deploy` — builds and starts server on port 6767
2. Open your configured CLIENT_URL — see pairing page
3. Scan QR with phone, complete pairing
4. Set PIN, verify PIN entry works
5. Send message, verify streaming response
6. Test cancel functionality

---
> Source: [jamierpond/claude-remote](https://github.com/jamierpond/claude-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

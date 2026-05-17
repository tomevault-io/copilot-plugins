## meetrec-desktop-client

> Handles infrastructure concerns only:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MeetRec Desktop Client is an Electron + React + TypeScript desktop application that continuously records meeting room audio, encodes it to WAV, and uploads segments to a MeetRec server with offline queuing and retry.

## Commands

```bash
# Development
npm run dev              # Start all dev processes concurrently
npm run dev:renderer     # Vite dev server only (http://127.0.0.1:5173)
npm run dev:electron     # esbuild watch for Electron files only

# Type checking & build
npm run typecheck        # TypeScript type check
npm run build            # Full build (typecheck + Vite + Electron bundle)

# Distribution (Windows)
npm run dist:portable    # Build portable .exe
npm run dist:setup       # Build NSIS installer
npm run dist:all         # Build both variants
```

There are no test commands in this project.

## Architecture

### Process Model
```
Electron Main (electron/main.ts)
    ↓ IPC via window.meetrecDesktop (electron/preload.ts)
React Renderer (src/app/AppShell.tsx)
    ↓ HTTP via axios
MeetRec Server (/api/v1/*)
```

### Electron Main Process (`electron/main.ts`)
Handles infrastructure concerns only:
- Settings persistence in SQLite (`userData/meetrec.db`)
- Upload queue persistence in SQLite (`segments` table) + WAV files in `userData/segments/`
- IPC handlers exposed via channels like `auth:*`, `settings:*`, `session:*`, `queue:*`, `diag:*`, `capture:*`

### IPC Bridge (`electron/preload.ts`)
Context-isolated bridge exposing `window.meetrecDesktop` to the renderer. TypeScript types live in `src/types/electron.d.ts`. The renderer should call `window.meetrecDesktop.*` methods directly (no extra wrapper).

### React Renderer
Entry point is `src/main.tsx` → `src/app/AppShell.tsx`. The renderer is UI + microphone capture only; all stateful business logic runs in Electron main.

```
src/
  app/AppShell.tsx              # Root component: routes to screens based on state/role
  hooks/
    useMainState.ts             # Subscribes to all main push events + invokes commands
    useCapture.ts               # AudioWorklet lifecycle; streams PCM to main
    useDevices.ts               # Microphone device enumeration
  screens/
    LoginScreen.tsx
    SetupWizard.tsx
    OperatorScreen.tsx
    AdminScreen.tsx
    ParticipantBlockedScreen.tsx
  components/
    RecorderStatus.tsx
    QueuePanel.tsx
    SettingsForm.tsx
    StartMeetingModal.tsx
```

### Role-Based Routing (`AppShell.tsx`)
After login, `getUserRole(user)` maps the server-side user object to one of three roles:
- `admin` → `AdminScreen` (full access: recorder + all settings tabs)
- `operator` → `OperatorScreen` (recorder only, no settings)
- `participant` → `ParticipantBlockedScreen` (no access)

### HTTP API
All HTTP calls happen in Electron main services:
- `POST /api/v1/auth/login` — returns JWT
- `GET /api/v1/auth/me` — session verification + user profile
- `POST /api/v1/inbox/upload` — multipart WAV upload
- `POST /api/v1/inbox/session/{session_id}/complete` — finalize session

### Data Flow
1. Renderer captures microphone input via AudioWorklet (`src/hooks/useCapture.ts`)
2. PCM chunks are sent as Transferable buffers to Electron main (`capture:pcm-chunk`)
3. Main accumulates PCM, encodes WAV, writes to disk, and inserts a `segments` row in SQLite
4. `UploadWorker` drains the queue and uploads segments with retry/backoff
5. Final segment triggers `/session/{id}/complete`; on completion, WAVs are deleted

## Key Design Notes

- **WAV encoding** happens in Electron main (`electron/audio/wav-encoder.ts`).
- **Segment duration** is configurable (30s–3600s) via settings.
- **Queue persistence** survives app restarts — uploads resume automatically on next launch.
- **Path alias**: `@/` maps to `src/` in both Vite and TypeScript configs.
- **Server reachability check** uses a probe request to `GET /auth/me` with a dummy token; a 401 or 403 response counts as \"server reachable\".

---
> Source: [StefanJoita/MeetRec_desktop-client](https://github.com/StefanJoita/MeetRec_desktop-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->

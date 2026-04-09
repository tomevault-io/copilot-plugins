## next-websocket-chat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server (custom HTTP + WebSocket on :3000)
pnpm build        # Production Next.js build
pnpm start        # Production server (NODE_ENV=production)
pnpm lint         # ESLint
pnpm format       # Prettier (write)
pnpm format:check # Prettier (check only)
```

No test framework is configured.

## Architecture

Real-time chat app with planning poker voting. Next.js 16 App Router frontend served by a custom Node.js HTTP server that also handles WebSocket connections.

### Server (`server/`)

- **`index.ts`** — Creates HTTP server from Next.js, attaches WebSocket server on `/ws`, handles REST endpoints (`/api/startSession`, `/api/profile?username=X`). Routes HMR upgrades to Next.js, all others to ws.
- **`chatRoom.ts`** — Manages sessions (Map of sessionId → socket/user/room/vote/profile), rooms (Map of roomName → Set of sessionIds), and broadcasting. Handles all WebSocket message types: connect, joinRoom, leaveRoom, sendMessage, userTyping, userVote, revealVotes, clearVoting, updateProfile.
- **`db.ts`** — SQLite (better-sqlite3, WAL mode) with single `user_profiles` table. Stored avatar configs are merged with defaults for forward compatibility when options are added.
- **`env.ts`** — Zod-validated environment variables (PORT, NODE_ENV).

### Client (`src/`)

- **`app/page.tsx`** — Login page. Reads room from URL hash for link sharing, persists name/room to localStorage.
- **`app/ChatRoom.tsx`** — Main chat UI (~575 lines). Header, message list, user sidebar with votes, vote controls, message input. Mobile handling via Visual Viewport API for iOS keyboard.
- **`app/AvatarEditorModal.tsx`** — Avatar editor with 20+ customizable BigHead properties, live preview, shuffle, color picker.
- **`lib/useChat.ts`** — Core WebSocket hook. Manages connection lifecycle, all state (messages, users, typing, votes), Zod-validates all incoming events.
- **`lib/types.ts`** — Shared TypeScript types. Discriminated unions for all WebSocket events and requests. Vote types use a string enum (UNVOTE, HIDDEN, QUESTION, BREAK, ZERO through TWENTY_ONE).
- **`lib/schemas.ts`** — Zod schemas mirroring all types, used for both client and server validation.

### Data flow

1. Client fetches `/api/startSession` → gets sessionId
2. Client opens WebSocket to `/ws`, sends `connect` with sessionId
3. Client sends `joinRoom` → server broadcasts user list to room
4. All messages/votes/typing go through WebSocket as JSON with discriminated `type` field
5. Profile changes persist to SQLite and broadcast to room

## Code Style

- TypeScript strict mode with `noUnusedLocals`, `noUnusedParameters`, `exactOptionalPropertyTypes`
- Prettier: double quotes, semicolons, 2-space indent, trailing commas, 80 char width, Tailwind class sorting
- Tailwind CSS 4 with CSS custom properties for theming (dark theme)
- React Compiler enabled via `next.config.ts`
- Fonts: Outfit (body), JetBrains Mono (monospace)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gordolio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gordolio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

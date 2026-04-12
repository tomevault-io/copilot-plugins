## minimal-chat

> A registration-free real-time chat app. Users pick a username (stored in sessionStorage), create or join rooms via random nanoid URLs, and chat over WebSockets. Messages are persisted in MongoDB.

# AI Chat — Nuxt 3 Real-Time Chat Application

## Project Overview

A registration-free real-time chat app. Users pick a username (stored in sessionStorage), create or join rooms via random nanoid URLs, and chat over WebSockets. Messages are persisted in MongoDB.

## Tech Stack

- **Framework**: Nuxt 3 with TypeScript
- **Frontend**: Vue 3 Composition API with TypeScript (`<script setup lang="ts">`), Tailwind CSS, @vueuse/core
- **Backend**: Nitro server routes + native WebSocket + TypeScript (CrossWS pub/sub)
- **Database**: MongoDB (native `mongodb` driver, no Mongoose)
- **ID Generation**: nanoid (12-char URL-friendly IDs)

## Architecture

### Data Flow
- REST API (`/api/rooms`, `/api/messages`) for CRUD operations
- WebSocket (`/ws/[roomId]`) for real-time messaging via CrossWS pub/sub
- Each room is a CrossWS channel — `peer.subscribe(roomId)` handles fan-out
- `peer.publish()` sends to all other subscribers; `peer.send()` echoes to sender

### State Management
- `useState()` for SSR-safe shared state
- `sessionStorage` for username + color persistence (per-tab)
- In-memory `Map<roomId, Set<peerId>>` for server-side participant tracking

### Database
- **Collections**: `rooms`, `messages`
- **Index**: `{ roomId: 1, createdAt: 1 }` on messages
- Room `_id` is a string (nanoid), not ObjectId — use `as any` cast on MongoDB queries

## Key Conventions

### Code Style
- 2-space indentation
- `<script setup lang="ts">` on all components
- Type imports: `import type { X } from '~/types'`
- Path alias `~` for project root
- No semicolons in most files
- Async/await over .then() chains

### Naming
- **Components**: PascalCase (`ChatMessage.vue`)
- **Composables**: `use` prefix, camelCase (`useUsername.ts`)
- **Server routes**: file-based (`[id].get.ts`, `index.post.ts`)
- **Types/Interfaces**: PascalCase (`WsServerPayload`)

### Styling
- Tailwind utility classes only — no custom CSS beyond directives
- Pastel color palette (16 colors) defined in `types/index.ts` as `USER_COLORS`

### Error Handling
- Server: `throw createError({ statusCode, statusMessage })`
- WebSocket: silent catch on malformed JSON
- Client: try-catch with user-facing error messages

## Important Files

| File | Purpose |
|------|---------|
| `types/index.ts` | All shared interfaces, WS payload types, USER_COLORS |
| `server/utils/mongodb.ts` | Singleton MongoDB connection (auto-imported by Nitro) |
| `server/routes/ws/[roomId].ts` | WebSocket handler — core real-time logic |
| `composables/useUsername.ts` | Username + color state with sessionStorage |
| `composables/useChatSocket.ts` | Client WebSocket wrapper (VueUse useWebSocket) |
| `pages/room/[id].vue` | Chat room page — merges history + live messages |
| `nuxt.config.ts` | WebSocket flag, Tailwind module, runtimeConfig |

## Commands

```bash
npm run dev        # Start dev server (localhost:3000)
npm run build      # Production build
npm run preview    # Preview production build
npx nuxi typecheck # Run TypeScript type checking
```

## Environment Variables

Set in `.env` (gitignored):
```
NUXT_MONGODB_URI=mongodb+srv://...   # Auto-maps to runtimeConfig.mongodbUri
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paseczkimihaly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paseczkimihaly)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

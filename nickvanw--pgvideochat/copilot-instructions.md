## pgvideochat

> Real-time chat and video/audio calling app built with SvelteKit + PostgreSQL.

# SpaceChatDB

Real-time chat and video/audio calling app built with SvelteKit + PostgreSQL.

## Architecture

- **Frontend**: SvelteKit app in `src/` — connects to pg-relay via WebSocket
- **Backend**: `pg-relay/` — Node.js WebSocket server that reads/writes PostgreSQL and uses logical replication to push changes to clients in real-time
- **Database**: PostgreSQL with `wal_level = logical` — schema in `pg-relay/schema.sql`

## Running

1. Set up PostgreSQL with `wal_level = logical`
2. Apply schema: `psql $DATABASE_URL < pg-relay/schema.sql`
3. Create `.env` in project root: `VITE_RELAY_URI=ws://localhost:9000`
4. Create `.env` in `pg-relay/`: `DATABASE_URL=postgresql://...`
5. Start pg-relay: `cd pg-relay && npm install && npm run dev`
6. Start frontend: `npm install && npm run dev`

## Key Files

- `pg-relay/src/server.ts` — WebSocket server, message routing
- `pg-relay/src/reducers.ts` — Business logic (chat, calls, media)
- `pg-relay/src/replication.ts` — PG logical replication → WebSocket push
- `pg-relay/src/db.ts` — Database pool and replication slot setup
- `pg-relay/schema.sql` — Full database schema
- `src/lib/relay.ts` — Client-side WebSocket connection and state management
- `src/lib/callRuntime.ts` — Audio/video capture and playback

---
> Source: [nickvanw/PgVideoChat](https://github.com/nickvanw/PgVideoChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->

## gg-retro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust implementation of the Gadu-Gadu 6.0 protocol server and client. Gadu-Gadu is a Polish instant messaging service. The project reimplements the legacy protocol to enable communication with original GG clients.

The codebase is organized as a Cargo workspace with three main components:
- `protocol/`: Core GG protocol implementation (codec, packet definitions, hash functions)
- `server/`: GG server with both TCP messenger and HTTP API endpoints
- `client/`: Simple test client for protocol validation

## Building and Running

Build the entire workspace:
```bash
cargo build
```

Run the server:
```bash
cargo run --bin gg-server
```

Run the test client:
```bash
cargo run --bin gg-client
```

The server listens on:
- Port 8074: GG messenger protocol (TCP)
- Port 80: HTTP API endpoints

## Database

The server uses SQLite with WAL mode. Database file: `./gg.db`

Migrations are in `server/migrations/` and run automatically on server startup via sqlx::migrate!().

Tables:
- `users`: User accounts (uin, password hash, email, etc.)
- `tokens`: CAPTCHA tokens for registration
- `messages`: Message queue for offline message delivery

## Architecture

### Protocol Layer (`protocol/`)

The `gg-protocol` crate provides the foundation:
- **GGCodec**: Tokio codec implementing Encoder/Decoder for the binary GG protocol
- **CodecMode**: Server vs Client mode (affects which packet types are decoded)
- **Packets**: All GG protocol packets (Login, Messages, Presence, etc.)
- **CP1250 encoding**: All text uses Windows-1250 character encoding
- **Hash**: GG login hash algorithm implementation

Key: The protocol uses little-endian byte order and fixed-size packet headers (type + length).

### Server Architecture (`server/`)

The server runs two concurrent tasks coordinated by CancellationToken:

1. **Messenger Server** (`messenger/mod.rs`): TCP server on port 8074
   - Accepts GG client connections
   - Spawns `UserSessionController` for each connection
   - Session lifecycle: establish_session() → sync() → run() → cleanup()

2. **HTTP Server** (`api/mod.rs`): HTTP API on port 80
   - `appmsg`: Server discovery endpoint (returns server IP)
   - `captcha`: CAPTCHA token generation and image rendering
   - `register`: Account registration
   - `sendpwd`: Password recovery
   - `ads`: Ad placeholder endpoint

### Shared State (`core.rs`)

`AppState` contains:
- `db_pool`: SQLite connection pool (32 max connections)
- `host_ip`: Server's local IP address
- `presence_hub`: Manages user online/offline status and broadcasts changes
- `message_dispatcher`: Handles message routing and offline message queuing

Both servers share the same `Arc<AppState>`.

### Session Management (`messenger/session.rs`)

`UserSessionController` manages individual client connections:
- **Handshake**: Send Welcome packet with random seed
- **Authentication**: Validate GG login hash against database
- **Sync**: Send initial contact presence and offline messages
- **Runtime**: Main event loop processing packets, presence changes, and incoming messages

The controller subscribes to two broadcast streams:
- `PresenceChangeStream`: Notifies when contacts come online/offline
- `MessagesStream`: Receives messages addressed to this user

### Presence System (`messenger/presence.rs`)

`PresenceHub` is a shared, thread-safe registry of online users:
- When users login, they register their presence and subscribe to changes
- When status changes, all subscribers receive updates
- Uses broadcast channels for efficient 1-to-N notification

### Message Dispatching (`messenger/messages.rs`)

`MessageDispatcher` routes messages between users:
- If recipient is online: delivers immediately via their MessagesStream
- If recipient is offline: stores in database for later delivery
- Messages are delivered during session sync phase

### Models and Repositories (`models/`)

Repository pattern for database access:
- `UserRepository`: User CRUD, authentication
- `TokenRepository`: CAPTCHA token management
- `MessageRepository`: Message persistence and retrieval

All repositories take `&DatabasePool` and return `Result<T, RepositoryError>`.

## Protocol Reference

Full protocol documentation is in `docs/protocol.md` (Polish language).

Key protocol concepts:
- **UIN**: User Identification Number (the user's GG number)
- **Seed**: Random value sent by server, used in login hash calculation
- **Login hash**: SHA1-based hash computed from password and seed
- **Contact list**: Users subscribe to presence updates for their contacts
- **Status types**: Available, Busy, Invisible, etc. (defined in `protocol/src/consts.rs`)

## Client Setup (from README)

To test with original GG clients, redirect these domains to 127.0.0.1:
```
127.0.0.1  appmsg.gadu-gadu.pl
127.0.0.1  appsrv.gadu-gadu.pl
```

Add to `C:\Windows\System32\drivers\etc\hosts` (Windows) or `/etc/hosts` (Linux/Mac).

Alternatively, use the patching script in `patch/` to modify the GG client executable.

## Common Patterns

**Adding a new packet type:**
1. Define packet structure in `protocol/src/packets/`
2. Add packet type constant to `protocol/src/consts.rs`
3. Implement encoding in `GGCodec::encode()` in `protocol/src/codec.rs`
4. Implement decoding in `GGCodec::decode()` in `protocol/src/codec.rs`
5. Handle packet in `UserSessionController::run()` in `server/src/messenger/session.rs`

**Adding a new HTTP endpoint:**
1. Create module in `server/src/api/`
2. Define router using `axum::Router`
3. Merge router in `server/src/api/mod.rs::router()`

**Database migrations:**
Place SQL files in `server/migrations/` with naming: `YYYYMMDDHHMMSS_description.sql`

---
> Source: [macbury/gg-retro](https://github.com/macbury/gg-retro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->

## selfsync

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**selfsync** — self-hosted Chrome sync solution. A Cargo workspace with two crates:

- **selfsync-server** — Chrome sync server (axum + sea-orm + SQLite). Handles `COMMIT` and `GET_UPDATES` via protobuf. User identity from protobuf `share` field.
- **selfsync-nigori** — Nigori encryption library (AES-128-CBC + HMAC-SHA256, PBKDF2/Scrypt key derivation).

## Build & Test

```bash
cargo build --release                        # Build all
cargo build --release -p selfsync-server     # Server only
cargo check                                  # Type check workspace
cargo clippy                                 # Lint check
cargo test                                   # Run tests
```

## Project Structure

```
selfsync/
├── crates/
│   ├── nigori/          # Nigori encryption library
│   │   └── src/
│   │       ├── lib.rs       # Nigori struct: encrypt/decrypt/get_key_name
│   │       ├── keys.rs      # PBKDF2 and Scrypt key derivation
│   │       ├── stream.rs    # NigoriStream binary serialization
│   │       └── error.rs     # Error types
│   └── sync-server/     # Chrome sync server
│       ├── proto/           # 92 Chromium .proto files
│       ├── build.rs         # prost-build proto compilation
│       └── src/
│           ├── main.rs      # axum server entry point
│           ├── proto.rs     # Generated protobuf types
│           ├── auth.rs      # Default email constant
│           ├── progress.rs  # Progress token encoding/decoding
│           ├── util.rs      # Shared utilities (gen_id, now_millis, etc.)
│           ├── db/
│           │   ├── mod.rs       # SQLite connection + WAL mode
│           │   ├── migration.rs # Schema creation (users, sync_entities)
│           │   └── entity/      # sea-orm entities
│           └── handler/
│               ├── sync.rs      # POST /command/ dispatch
│               ├── commit.rs    # COMMIT: create/update entities
│               ├── get_updates.rs # GET_UPDATES: fetch by version
│               ├── init.rs      # User initialization (Nigori + bookmarks)
│               └── users.rs     # GET / user list page
```

## Sync Server

- **Endpoint**: `POST /command/` — handles protobuf `ClientToServerMessage` → `ClientToServerResponse`
- **Alternate**: `POST /chrome-sync/command/` — same handler, for `--sync-url=http://host:port/chrome-sync`
- **Dashboard**: `GET /` — HTML user list
- **Auth**: reads email from protobuf `share` field (Chrome always sends the signed-in account email); fallback `anonymous@localhost`
- **Storage**: SQLite (WAL mode), single `sync_entities` table (no sharding)
- **Version**: per-user monotonic counter (`users.next_version`), assigned on commit
- **Progress tokens**: `v1,{data_type_id},{version}` base64-encoded
- **Config env vars**: `SELFSYNC_DB` (default: `selfsync.db`), `SELFSYNC_ADDR` (default: `127.0.0.1:8080`)
- **User init**: on first sync, auto-creates Nigori node (keystore passphrase) + 4 bookmark permanent folders
- **Proto module**: `proto.rs` wraps generated code with `#[allow(clippy::all, dead_code, deprecated)]`

## Chrome Sync Protocol Gotchas

- `--sync-url=http://host:port` — Chrome appends `/command/` automatically; do NOT include it in the URL
- `ClientToServerMessage.share` contains the user's email — no need for external auth/headers
- `ClientToServerResponse.error_code` must be explicitly set to `SUCCESS (0)` — proto default is `UNKNOWN`, Chrome treats it as error
- `NigoriSpecifics.passphrase_type`: `KEYSTORE_PASSPHRASE = 2`, `CUSTOM_PASSPHRASE = 4` — wrong value causes "Needs passphrase" error
- Chrome caches Nigori state locally; after server DB reset, must use fresh Chrome profile (`--user-data-dir=/tmp/test`)
- NEW_CLIENT GetUpdates expects Nigori entity to exist on server; without it Chrome stalls at "Initializing"
- GetUpdates response must include `encryption_keys` when `need_encryption_key=true` and origin is `NEW_CLIENT`
- prost generates `EntitySpecifics.specifics_variant` (oneof), not individual fields like `bookmark`/`nigori`
- Proto field `client_tag_hash` (not `client_defined_unique_tag`), `message_contents` is `i32` (not enum)
- Chromium proto imports use `components/sync/protocol/` prefix — must strip when copying to local `proto/` dir

## Key Chromium Source References

Relevant paths in `~/modous/chromium/src/`:

- `components/sync/base/sync_util.cc` — `GetSyncServiceURL()`, reads `--sync-url` (no branding guard)
- `components/sync/engine/sync_manager_impl.cc` — `MakeConnectionURL()`, appends `/command/` path
- `components/sync/engine/net/url_translator.cc` — `AppendSyncQueryString()`, adds `client` and `client_id` params
- `components/sync/engine/net/http_bridge.cc` — `MakeAsynchronousPost()`, HTTP request construction
- `components/sync/protocol/sync.proto` — `ClientToServerMessage`, `ClientToServerResponse`
- `components/sync/engine/loopback_server/loopback_server.cc` — Reference sync server implementation

---
> Source: [loyalpartner/selfsync](https://github.com/loyalpartner/selfsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->

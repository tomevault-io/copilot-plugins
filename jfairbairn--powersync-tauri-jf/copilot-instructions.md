## powersync-tauri-jf

> WebKit-based apps (Tauri on macOS/iOS, Electron on macOS, etc.) suffer from severe CPU usage when running SQLite via WASM. The standard `@powersync/web` package uses WASM SQLite which causes the CPU to spin at 100%+ even when idle.

# PowerSync Tauri Plugin Development Guide

## Why This Exists

WebKit-based apps (Tauri on macOS/iOS, Electron on macOS, etc.) suffer from severe CPU usage when running SQLite via WASM. The standard `@powersync/web` package uses WASM SQLite which causes the CPU to spin at 100%+ even when idle.

This plugin solves that by running SQLite natively via Rust (rusqlite), providing the same PowerSync API but without the WASM overhead. **It's a drop-in replacement for `@powersync/web`** - you change your import from `@powersync/web` to `@jfairbairn/tauri-plugin-powersync-jf` and everything else stays the same.

## Project Overview

This is `tauri-plugin-powersync-jf`, a Tauri 2.0 plugin that provides SQLite database functionality using native Rust SQLite (rusqlite) with full PowerSync sync support.

## Status

**Production Ready** - All core functionality is implemented and tested.

- Native SQLite via rusqlite with bundled SQLite
- PowerSync extension loaded from `deps/powersync-sqlite-core`
- Full DBAdapter interface implementation (execute, getAll, getOptional, get, executeBatch)
- Transactions (read and write) with proper commit/rollback
- TypeScript bindings matching `@powersync/web` API
- Full sync support via `PowerSyncDatabase`
- CRUD tracking (PUT, PATCH, DELETE operations)
- Schema replacement with views
- Reactive queries via `watch()`
- Offline-first with automatic sync on reconnect

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│ PowerSync Backend Service                                   │
└─────────────────────────────────────────────────────────────┘
                              ↕
                    [TauriRemote / AbstractRemote]
                              ↕
┌─────────────────────────────────────────────────────────────┐
│ TauriStreamingSyncImplementation                            │
│  └─ Manages connection, streams data, triggers uploads      │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│ SqliteBucketStorage (from @powersync/common)                │
│  └─ Manages ps_crud, ps_oplog, ps_buckets via DBAdapter     │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│ TauriDBAdapter → Tauri IPC → Native rusqlite                │
│  └─ PowerSync extension loaded (.dylib/.so/.dll)            │
└─────────────────────────────────────────────────────────────┘
```

## Key Files

| File | Purpose |
|------|---------|
| `src/lib.rs` | Plugin entry, registers commands |
| `src/commands.rs` | Tauri command handlers (including PowerSync ops) |
| `src/database.rs` | rusqlite connection management |
| `src/extension.rs` | PowerSync extension loading |
| `src/error.rs` | Error types |
| `guest-js/TauriDBAdapter.ts` | DBAdapter implementation |
| `guest-js/TauriPowerSyncDatabase.ts` | Simple database (no sync) |
| `guest-js/TauriPowerSyncDatabaseFull.ts` | Full sync database |
| `guest-js/TauriStreamingSyncImplementation.ts` | Sync stream implementation |
| `deps/powersync-sqlite-core/` | Git submodule with extension source |

## Development Commands

```bash
# Build Rust plugin (also builds PowerSync extension)
cargo build

# Build JS bindings
npm run build

# Run test app
npm run test:tauri:dev

# Format code
cargo fmt && npm run format
```

## Usage

### Full Sync (Recommended)

```typescript
import { PowerSyncDatabase } from '@jfairbairn/tauri-plugin-powersync-jf';
import { Schema, Table, column } from '@powersync/common';

const schema = new Schema({
  todos: new Table({
    description: column.text,
    completed: column.integer,
  }),
});

const db = new PowerSyncDatabase({
  database: 'myapp',
  schema,
});

await db.init();

// Connect to PowerSync service
await db.connect(myBackendConnector);

// Use like a normal database - changes sync automatically
await db.execute('INSERT INTO todos (id, description) VALUES (uuid(), ?)', ['Buy milk']);
const todos = await db.getAll('SELECT * FROM todos');
```

### Simple Database (No Sync)

```typescript
import { TauriPowerSyncDatabase } from '@jfairbairn/tauri-plugin-powersync-jf';

const db = new TauriPowerSyncDatabase({ database: 'myapp' });
await db.init();

await db.execute('CREATE TABLE IF NOT EXISTS todos (id TEXT PRIMARY KEY, title TEXT)');
await db.execute('INSERT INTO todos (id, title) VALUES (?, ?)', ['1', 'Buy milk']);

const todos = await db.getAll('SELECT * FROM todos');
```

## Testing in a Tauri App

1. Add to your app's `src-tauri/Cargo.toml`:
   ```toml
   [dependencies]
   tauri-plugin-powersync-jf = { path = ".." }
   ```

2. Register in `src-tauri/src/lib.rs`:
   ```rust
   tauri::Builder::default()
       .plugin(tauri_plugin_powersync_jf::init())
   ```

3. Add permissions to `src-tauri/capabilities/default.json`:
   ```json
   { "permissions": ["powersync-jf:default"] }
   ```

4. Bundle the PowerSync extension in your app's resources (built automatically during `cargo build`)

## PowerSync Extension

The extension is built from `deps/powersync-sqlite-core` during `cargo build`. It provides:

- `powersync_init()` - Initialize PowerSync
- `powersync_replace_schema(json)` - Set up sync schema (creates views)
- `powersync_control(op, payload)` - Sync control operations
- `powersync_rs_version()` - Get extension version
- Internal tables: `ps_data_*`, `ps_crud`, `ps_buckets`, `ps_oplog`

### Cross-Platform Extension Files

| Platform | Extension File |
|----------|---------------|
| macOS | `libpowersync.dylib` |
| Linux | `libpowersync.so` |
| Windows | `powersync.dll` |

## DBAdapter Interface

TauriDBAdapter implements @powersync/common's DBAdapter plus PowerSync extension methods:

```typescript
interface TauriDBAdapter extends DBAdapter {
  // Standard DBAdapter methods
  execute(sql, params?): Promise<QueryResult>;
  getAll<T>(sql, params?): Promise<T[]>;
  readTransaction<T>(fn): Promise<T>;
  writeTransaction<T>(fn): Promise<T>;
  // ... etc

  // PowerSync extension methods
  isPowerSyncLoaded(): Promise<boolean>;
  getPowerSyncVersion(): Promise<string>;
  replaceSchema(schemaJson: string): Promise<void>;
  powerSyncControl(op: string, payload: string): Promise<string>;
  getCrudBatch(limit?: number): Promise<CrudEntry[]>;
  removeCrud(crudId: number): Promise<void>;
  hasPendingCrud(): Promise<boolean>;
  getWriteCheckpoint(): Promise<string | null>;
}
```

## Common Issues

### "Database not initialized"
Call `db.init()` before using the database.

### "PowerSync not initialized"
The PowerSync extension wasn't found or failed to load. Check that:
- The extension file exists in the app's resources
- The extension was built correctly during `cargo build`

### Transaction errors
Transactions use IDs passed across IPC. Each transaction must be committed or rolled back before starting another.

### Build errors on different platforms
The bundled SQLite feature in rusqlite compiles SQLite from source. Ensure you have a C compiler (gcc, clang, or MSVC).

## Test App

The `test-app/` directory contains a Tauri application for running the plugin test suite in a native context.

### Running Tests

```bash
npm run test:tauri:dev
```

This opens a window showing test results. The tests run automatically on load.

### Test Categories

The test suite verifies:

**DBAdapter Interface (15 tests)**
- Query execution (execute, getAll, getOptional, get)
- Batch operations (executeBatch)
- Transactions (readTransaction, writeTransaction, commit, rollback)
- Locks (readLock, writeLock)
- Return values (rowsAffected, insertId)

**Schema Operations (2 tests)**
- Creating tables from PowerSync schema
- Column type handling

**CRUD Operations (8 tests)**
- INSERT, UPDATE, DELETE operations
- INSERT OR REPLACE
- Batch inserts
- Transaction grouping
- Visibility of uncommitted changes within transactions

**Data Type Handling (5 tests)**
- NULL, INTEGER, REAL, BOOLEAN values
- Large integers (up to MAX_SAFE_INTEGER)

**PowerSync Extension (5 tests)**
- Extension loading verification
- Schema replacement (creates views)
- CRUD entry tracking (ps_crud table)
- Control commands

**Full Sync (10 tests)** - requires local backend
- Sync status transitions (connect, hasSynced, disconnect)
- Initial sync downloads data
- CRUD uploads (PUT, PATCH, DELETE)
- Reconnection preserves data
- Offline writes sync on reconnect
- Reactive queries (watch)
- Transaction batching

### Local Backend for Sync Tests

The `deps/self-host-demo/` directory contains a local PowerSync backend for testing sync functionality:

```bash
# Start the local backend
cd deps/self-host-demo/demos/nodejs
docker compose up -d

# Run tests (sync tests will now pass)
cd ../../../..
npm run test:tauri:dev
```

Without the backend running, the 10 sync tests are skipped (they show as "backend not available").

## References

- [PowerSync JS SDK](https://github.com/powersync-ja/powersync-js)
- [powersync-sqlite-core](https://github.com/powersync-ja/powersync-sqlite-core)
- [Tauri Plugin Development](https://v2.tauri.app/develop/plugins/)
- [rusqlite docs](https://docs.rs/rusqlite)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jfairbairn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jfairbairn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

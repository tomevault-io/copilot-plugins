## strata

> This skill enables AI coding agents to use, extend, and test the

# Strata — AI Agent Skill

## Purpose

This skill enables AI coding agents to use, extend, and test the
`github.com/AndrewDonelson/strata` package correctly and efficiently.
Strata is a **four-tier auto-caching data library for Go** that unifies
in-memory cache (L1), Redis (L2), PostgreSQL (L3), and an optional
peer-to-peer distributed ledger (L4) behind a single API.

---

## Package identity

```
import "github.com/AndrewDonelson/strata"
```

Minimum requirements: Go 1.21+, PostgreSQL 14+, Redis 6+.
The package is `package strata`.
Internal sub-packages (`internal/l1`, `internal/l2`, `internal/l3`,
`internal/l4`, `internal/codec`, `internal/clock`, `internal/metrics`)
are **not** part of the public API — never import them in application code.
(Exception: `internal/l4` may be imported directly when you need the
distributed sync layer; see the L4 section below.)

---

## Read tier behaviour

```
Get(ctx, schema, id, &dest)
  │
  ├─► L1 hit?  → return immediately           (~100 ns)
  ├─► L2 hit?  → populate L1 → return         (~500 µs)
  └─► L3 hit?  → populate L2, populate L1 → return  (~5 ms)
               └─► not found → ErrNotFound
```

Writes flow in the **opposite** direction determined by `WriteMode`.

---

## Lifecycle — exactly four steps

```go
// 1. Create
ds, err := strata.NewDataStore(strata.Config{
    PostgresDSN: "postgres://user:pass@host:5432/mydb",
    RedisAddr:   "localhost:6379",
})
if err != nil { /* handle */ }
defer ds.Close() // ← always defer Close; it flushes write-behind and stops workers

// 2. Register (once per schema, at startup, before any data operation)
err = ds.Register(strata.Schema{Name: "players", Model: &Player{}})

// 3. Migrate (idempotent — safe to call on every startup)
err = ds.Migrate(ctx)

// 4. Use
_ = ds.Set(ctx, "players", "p1", &Player{...})
```

---

## Struct definition rules

- Every model **must** have exactly one field tagged `strata:"primary_key"`.
  It must be of type `string`. The field name is conventionally `ID`.
- The model **must** be a concrete struct — not a map, interface, or slice.
- Pass a **pointer** to the struct as `Schema.Model` and as `dest` in `Get`.
- The `Name` field in `Schema` defaults to the snake_case struct name if omitted.

```go
type Player struct {
    ID        string    `strata:"primary_key"`
    Username  string    `strata:"unique,index"`
    Email     string    `strata:"index,nullable"`
    Level     int       `strata:"default:1"`
    Score     int64
    APIKey    string    `strata:"encrypted"`    // AES-256-GCM, string fields only
    Password  string    `strata:"omit_cache"`   // Postgres only; never in L1/L2
    Notes     string    `strata:"nullable"`
    CreatedAt time.Time `strata:"auto_now_add"` // set once on first write
    UpdatedAt time.Time `strata:"auto_now"`     // updated on every write
    Internal  string    `strata:"-"`            // completely ignored
}
```

### Full tag reference

| Tag | Effect |
|-----|--------|
| `primary_key` | Required. Identity field for Get/Set routing. |
| `unique` | UNIQUE constraint in Postgres. |
| `index` | Non-unique Postgres index. |
| `nullable` | Column allows NULL (default is NOT NULL). |
| `omit_cache` | Excluded from L1 **and** L2; Postgres only. |
| `omit_l1` | Excluded from L1 only; still in L2. |
| `default:X` | SQL `DEFAULT X` in the DDL. |
| `auto_now_add` | Set to `time.Now()` on first insert, never updated. |
| `auto_now` | Set to `time.Now()` on every write. |
| `encrypted` | AES-256-GCM encrypted at rest. Requires `EncryptionKey` in Config. String fields only. Stored plaintext in L1/L2 for read speed. |
| `-` | Completely ignored — not persisted, not cached. |

Multiple tags are **comma-separated** with no spaces: `strata:"unique,index,nullable"`.

### Go → Postgres type mapping

| Go | PostgreSQL |
|----|-----------|
| `string` | `TEXT` |
| `int`, `int32`, `int64` | `BIGINT` |
| `float32`, `float64` | `DOUBLE PRECISION` |
| `bool` | `BOOLEAN` |
| `time.Time` | `TIMESTAMPTZ` |
| `[]byte` | `BYTEA` |
| struct / map / slice | `JSONB` |

---

## `Schema` struct — all fields

```go
type Schema struct {
    Name      string         // table/collection name (defaults to snake_case struct name)
    Model     any            // *StructType — pointer to zero value of model
    L1        MemPolicy      // in-memory cache settings
    L2        RedisPolicy    // Redis cache settings
    L3        PostgresPolicy // Postgres persistence settings
    WriteMode WriteMode      // WriteThrough | WriteBehind | WriteThroughL1Async
    Indexes   []Index        // extra database indexes
    Hooks     SchemaHooks    // lifecycle callbacks
}

type MemPolicy struct {
    TTL        time.Duration  // 0 = use Config.DefaultL1TTL
    MaxEntries int            // 0 = use Config.L1Pool.MaxEntries; PER SHARD (256 shards)
    Eviction   EvictionPolicy // EvictLRU (default) | EvictLFU | EvictFIFO
}

type RedisPolicy struct {
    TTL       time.Duration // 0 = use Config.DefaultL2TTL
    KeyPrefix string        // optional; defaults to schema name
}

type PostgresPolicy struct {
    TableName   string // optional; defaults to schema name
    ReadReplica string // optional DSN for a read-only replica
    PartitionBy string // optional column for table partitioning
}

type Index struct {
    Fields []string // column names
    Unique bool
    Name   string   // optional; auto-generated if empty
}
```

---

## `Config` struct — all fields with defaults

```go
type Config struct {
    // Connections (no defaults — set via environment variables)
    PostgresDSN   string
    RedisAddr     string
    RedisPassword string
    RedisDB       int

    // Pool sizes — defaults applied by NewDataStore if zero
    L1Pool L1PoolConfig{ MaxEntries: 100_000, Eviction: EvictLRU }
    L2Pool L2PoolConfig{ /* zero values use go-redis defaults */ }
    L3Pool L3PoolConfig{ MaxConns: 20, MinConns: 2,
                         MaxConnLifetime: 30*time.Minute,
                         MaxConnIdleTime: 10*time.Minute }

    // TTL defaults (per-schema TTLs override these)
    DefaultL1TTL time.Duration // default: 5m
    DefaultL2TTL time.Duration // default: 30m

    // Write behaviour
    DefaultWriteMode          WriteMode     // default: WriteThrough
    WriteBehindFlushInterval  time.Duration // default: 500ms
    WriteBehindFlushThreshold int           // default: 100
    WriteBehindMaxRetry       int           // default: 5

    // Multi-node invalidation
    InvalidationChannel string // default: "strata:invalidate"

    // Pluggable components (nil → sensible no-op defaults applied)
    Codec   codec.Codec            // default: MsgPack (NOT JSON — see note below)
    Clock   clock.Clock            // default: real wall clock
    Metrics metrics.MetricsRecorder // default: no-op
    Logger  Logger                  // default: no-op

    // Encryption — nil = disabled; must be exactly 32 bytes when set
    EncryptionKey []byte
}
```

> **IMPORTANT:** The default `Codec` is **MsgPack**, not JSON.
> Values stored in Redis are MsgPack-encoded by default.
> If you need human-readable Redis keys for debugging, set `Codec: codec.JSON{}`.

---

## Full public API

### DataStore construction

```go
ds, err := strata.NewDataStore(cfg Config) (*DataStore, error)
ds.Close() error  // idempotent; always defer this
ds.Stats() Stats  // snapshot of Gets/Sets/Deletes/Errors/L1Entries/DirtyCount
strata.Version() string  // "YYYY.MM.DD-HHMM-env"
```

### Schema registration

```go
ds.Register(s Schema) error
// Returns ErrSchemaDuplicate if called twice with the same Name.
// Must be called before any data operation that references the schema.
```

### CRUD

```go
// Read
ds.Get(ctx, schemaName, id string, dest any) error
// Generic helper — preferred when T is known at compile time
p, err := strata.GetTyped[Player](ctx, ds, "players", "p1")

// Write
ds.Set(ctx, schemaName, id string, value any) error
ds.SetMany(ctx, schemaName string, pairs map[string]any) error  // id → *Model

// Delete (removes from all tiers)
ds.Delete(ctx, schemaName, id string) error
```

### Search

```go
// Standard — queries L3, populates L1/L2 as side-effect
ds.Search(ctx, schemaName string, q *Query, destSlice any) error
// q == nil means "all rows" up to the default limit (100)
// q must be a pointer to a Query value, e.g.: q := ...; ds.Search(ctx, name, &q, &results)

// Generic helper
q := strata.Q().Where("level > $1", 10).Build()
players, err := strata.SearchTyped[Player](ctx, ds, "players", &q)

// Cached — caches full result set in L2 by SQL fingerprint
ds.SearchCached(ctx, schemaName string, q *Query, destSlice any) error
```

### Exists & Count

```go
ok, err := ds.Exists(ctx, schemaName, id string) (bool, error)
// Checks L1 → L2 → L3 in order; returns true on first hit.

n, err := ds.Count(ctx, schemaName string, q *Query) (int64, error)
// Always hits L3. q == nil counts all rows.
```

### Cache control

```go
ds.Invalidate(ctx, schemaName, id string) error      // remove one key from L1+L2
ds.InvalidateAll(ctx, schemaName string) error        // flush all keys for a schema
ds.WarmCache(ctx, schemaName string, limit int) error // pre-load L3 → L1+L2 (0 = all rows)
ds.FlushDirty(ctx) error                              // force write-behind flush now
```

### Migrations

```go
ds.Migrate(ctx) error                     // apply DDL for all registered schemas
ds.MigrateFrom(ctx, dir string) error     // apply NNN_description.sql files from dir
records, err := ds.MigrationStatus(ctx)   // []MigrationRecord — inspect applied state
```

Migrations are **additive only** — Strata never drops or renames columns.
Destructive changes must use manual SQL files with `MigrateFrom`.

### Transactions

```go
err := ds.Tx(ctx).
    Set("players", "p1", &Player{...}).
    Set("scores",  "p1", &Score{...}).
    Delete("sessions", "old-sid").
    Commit()
// L3 is committed atomically; L1/L2 are updated only after a successful commit.
// Returns ErrTxFailed (wrapping the Postgres error) on rollback.
```

---

## Query builder

```go
q := strata.Q().
    Where("level > $1 AND region = $2", 10, "eu-west").
    OrderBy("score").Desc().
    Limit(25).Offset(50).
    Fields("id", "username", "score").  // column projection
    ForceL3().                           // bypass L1+L2 entirely
    Build()                              // returns Query (value type)

// Pass as pointer to Search/SearchTyped/Count:
ds.Search(ctx, "players", &q, &results)

// Inline (Build returns a value; take its address):
q2 := strata.Q().Where("level > $1", 10).Limit(50).Build()
ds.Search(ctx, "players", &q2, &results)

// Do NOT share a *Query across goroutines — Query is a value type; copy it first.
```

Passing `nil` as `*Query` is always valid and means "no filter, default limit".

---

## Error handling — always use `errors.Is`

```go
err := ds.Get(ctx, "players", id, &p)
switch {
case errors.Is(err, strata.ErrNotFound):
    // record does not exist in any tier
case errors.Is(err, strata.ErrSchemaNotFound):
    // Register was not called for this schema name
case errors.Is(err, strata.ErrL3Unavailable):
    // Postgres is down
case err != nil:
    // unexpected error
}
```

### Full error catalogue

```go
// Schema registration
strata.ErrSchemaNotFound    // schema name not registered
strata.ErrSchemaDuplicate   // Register called twice for same name
strata.ErrNoPrimaryKey      // struct has no primary_key tag
strata.ErrInvalidModel      // nil or non-pointer model
strata.ErrMissingPrimaryKey // struct passed to Set has zero/empty PK field

// Data
strata.ErrNotFound     // record absent from all tiers
strata.ErrDecodeFailed // codec or decryption error on read
strata.ErrEncodeFailed // codec or encryption error on write

// Infrastructure
strata.ErrL1Unavailable // L1 not initialised
strata.ErrL2Unavailable // Redis unavailable
strata.ErrL3Unavailable // Postgres unavailable
strata.ErrUnavailable   // called after Close()

// Transactions
strata.ErrTxFailed  // Postgres rolled back — original error is wrapped
strata.ErrTxTimeout // transaction deadline exceeded

// Configuration
strata.ErrInvalidConfig // missing required fields

// Hooks
strata.ErrHookPanic // BeforeSet/BeforeGet hook panicked (recovered)

// Write-behind
strata.ErrWriteBehindMaxRetry // dirty entry exhausted retry budget
```

---

## Write modes

| Mode | When to use |
|------|-------------|
| `WriteThrough` (default) | Normal CRUD where durability matters. L3 written before returning. |
| `WriteThroughL1Async` | L3+L2 written synchronously; L1 populated lazily on next read. Good when L1 hit rate is low. |
| `WriteBehind` | High-frequency writes (scores, counters). L3 written asynchronously in batches. Risk: data loss if process crashes before flush. |

Set per-schema or globally:

```go
strata.Schema{
    Name:      "leaderboard",
    Model:     &Score{},
    WriteMode: strata.WriteBehind,
}

// or globally:
strata.Config{DefaultWriteMode: strata.WriteBehind}
```

---

## Encryption

```go
key := make([]byte, 32)
if _, err := rand.Read(key); err != nil { ... }

ds, _ := strata.NewDataStore(strata.Config{
    EncryptionKey: key, // enables AES-256-GCM for all fields tagged `encrypted`
})
```

- Only `string` fields support `encrypted`.
- Plaintext is stored in L1/L2 (fast path); ciphertext only in Postgres.
- The nonce is randomly generated per write and prepended to the ciphertext.
- If `EncryptionKey` is not set, `encrypted` tags are silently ignored (no error).

---

## Lifecycle hooks

```go
strata.Schema{
    Name:  "users",
    Model: &User{},
    Hooks: strata.SchemaHooks{
        BeforeSet: func(ctx context.Context, v any) error {
            u := v.(*User)
            if u.Email == "" {
                return errors.New("email required")
            }
            return nil
        },
        AfterSet: func(ctx context.Context, v any) {
            // fire domain event, update search index, etc.
        },
        OnWriteError: func(ctx context.Context, key string, err error) {
            // write-behind entry exhausted retries — alert here
        },
    },
}
```

`BeforeSet` returning a non-nil error **aborts the write** — no tiers are touched.
`AfterSet` fires only after a fully successful write.

---

## Observability

### Logger

```go
type Logger interface {
    Info(msg string, keysAndValues ...any)
    Warn(msg string, keysAndValues ...any)
    Error(msg string, keysAndValues ...any)
    Debug(msg string, keysAndValues ...any)
}
```

Wrapping `log/slog`:

```go
type SlogAdapter struct{ L *slog.Logger }
func (a SlogAdapter) Info(m string, kv ...any)  { a.L.Info(m, kv...) }
func (a SlogAdapter) Warn(m string, kv ...any)  { a.L.Warn(m, kv...) }
func (a SlogAdapter) Error(m string, kv ...any) { a.L.Error(m, kv...) }
func (a SlogAdapter) Debug(m string, kv ...any) { a.L.Debug(m, kv...) }
```

### Stats snapshot

```go
s := ds.Stats()
// s.Gets, s.Sets, s.Deletes, s.Errors int64
// s.L1Entries  int64 — current in-memory entry count
// s.DirtyCount int64 — write-behind entries pending flush
```

---

## Testing without a database

All unit tests use white-box injection — set `ds.l3` (unexported) to a mock
that implements the private `l3Backend` interface, or simply create a
`DataStore` with an empty `Config{}` (no DSN = no Postgres, no Redis).

### Pattern 1 — no external services

```go
func TestMyFeature(t *testing.T) {
    ds, err := strata.NewDataStore(strata.Config{}) // L1 only, no Postgres, no Redis
    require.NoError(t, err)
    defer ds.Close()

    require.NoError(t, ds.Register(strata.Schema{
        Name:  "items",
        Model: &MyModel{},
    }))
    // L3 operations will return ErrL3Unavailable — expected in unit tests.
}
```

### Pattern 2 — mock Postgres (white-box, same package)

```go
// In package strata (white-box test file):
type errL3 struct{ err error }
func (e *errL3) Upsert(_ context.Context, _ string, _ []string, _ []any, _ string) error { return e.err }
func (e *errL3) DeleteByID(_ context.Context, _, _ string, _ any) error { return e.err }
func (e *errL3) Query(_ context.Context, _ string, _ []any) (pgx.Rows, error) { return nil, e.err }
func (e *errL3) QueryRow(_ context.Context, _ string, _ []any) pgx.Row { return &errRow{e.err} }
func (e *errL3) Exec(_ context.Context, _ string, _ []any) error { return e.err }
func (e *errL3) Exists(_ context.Context, _, _ string, _ any) (bool, error) { return false, e.err }
func (e *errL3) Count(_ context.Context, _, _ string, _ []any) (int64, error) { return 0, e.err }
func (e *errL3) BeginTx(_ context.Context) (pgx.Tx, error) { return nil, e.err }
func (e *errL3) Close() {}

ds.l3 = &errL3{err: strata.ErrL3Unavailable}
```

### Pattern 3 — integration tests with Testcontainers

```go
//go:build integration
package strata_test

import (
    "testing"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
)

func TestIntegration(t *testing.T) {
    pgc, _ := postgres.Run(ctx, "postgres:15")
    defer pgc.Terminate(ctx)
    dsn, _ := pgc.ConnectionString(ctx, "sslmode=disable")
    ds, _ := strata.NewDataStore(strata.Config{PostgresDSN: dsn})
    defer ds.Close()
    ...
}
```

---

## Common patterns

### Service initialization

```go
func NewPlayerService(cfg strata.Config) (*PlayerService, error) {
    ds, err := strata.NewDataStore(cfg)
    if err != nil {
        return nil, fmt.Errorf("strata init: %w", err)
    }
    if err := ds.Register(strata.Schema{
        Name:  "players",
        Model: &Player{},
        L1:    strata.MemPolicy{TTL: time.Minute, MaxEntries: 10_000},
        L2:    strata.RedisPolicy{TTL: 30 * time.Minute},
    }); err != nil {
        return nil, err
    }
    if err := ds.Migrate(context.Background()); err != nil {
        return nil, err
    }
    return &PlayerService{ds: ds}, nil
}
```

### Paged search

```go
func (s *PlayerService) ListTopPlayers(ctx context.Context, page, size int) ([]Player, error) {
    q := strata.Q().
        OrderBy("score").Desc().
        Limit(size).Offset(page*size).
        Build()
    return strata.SearchTyped[Player](ctx, s.ds, "players", &q)
}
```

### Upsert pattern

```go
// ds.Set always upserts (INSERT … ON CONFLICT DO UPDATE in Postgres).
// If the record may or may not already exist, just call Set.
err := ds.Set(ctx, "players", player.ID, player)
```

### Conditional write (check-then-set)

```go
// Strata has no native CAS. Use Exists + Set.
// WARNING: not atomic — suitable only for low-contention scenarios.
exists, err := ds.Exists(ctx, "players", id)
if err != nil { return err }
if !exists {
    return ds.Set(ctx, "players", id, &newPlayer)
}
return strata.ErrSchemaDuplicate // or a domain error
```

### Bulk load at startup

```go
if err := ds.WarmCache(ctx, "config", 0); err != nil { // 0 = all rows
    log.Warn("cache warm failed — first reads will hit Postgres", "err", err)
}
```

### Atomic multi-schema write

```go
err := ds.Tx(ctx).
    Set("orders",   order.ID,   order).
    Set("inventory", item.ID,   item).
    Delete("carts", cartID).
    Commit()
if errors.Is(err, strata.ErrTxFailed) {
    // all operations rolled back; safe to retry
}
```

---

## Anti-patterns and mistakes to avoid

| Wrong | Right |
|-------|-------|
| `Register` after `Set`/`Get` | Always `Register` before data operations |
| Ignoring `Close()` | Always `defer ds.Close()` after a successful `NewDataStore` |
| Calling `Migrate` before `Register` | `Migrate` acts on registered schemas — register first |
| Sharing a `*Query` across goroutines | `Query` is a value type; copy or build a new one per goroutine |
| Hardcoding `EncryptionKey` | Load from env / secrets manager  |
| Using `omit_cache` for frequently-read fields | Use `omit_cache` only for write-only / rarely-read fields like password hashes |
| `MaxEntries` set globally thinking it limits total memory | `MaxEntries` is **per-shard** (256 shards). For ~50 000 total limit, set `MaxEntries: 200`. |
| Passing `*T` where `T` is not the model type to `Get` | `dest` must be `*ModelType` matching the registered schema |
| Calling `MigrateFrom` with destructive SQL | Strata does not prevent destructive DDL via `MigrateFrom` — review files carefully |
| Assuming `SearchCached` invalidates automatically | `SearchCached` caches by SQL fingerprint and expires by TTL only — invalidate manually with `Invalidate` or `InvalidateAll` if data changes frequently |
| Not handling `ErrTxFailed` as retriable | `ErrTxFailed` is almost always a serialization failure or conflict — safe to retry with backoff |

---

## Configuration checklist for production

```go
strata.Config{
    PostgresDSN: os.Getenv("POSTGRES_DSN"),
    RedisAddr:   os.Getenv("REDIS_ADDR"),

    L1Pool: strata.L1PoolConfig{
        MaxEntries: 200,          // per-shard; tune for model size vs available RAM
        Eviction:   strata.EvictLRU,
    },
    L3Pool: strata.L3PoolConfig{
        MaxConns: 30,             // match your Postgres max_connections budget
        MinConns: 5,
    },

    DefaultL1TTL: 2 * time.Minute,
    DefaultL2TTL: time.Hour,

    EncryptionKey: loadKeyFromVault(), // 32 bytes; nil disables encryption

    Logger:  mySlogAdapter,
    Metrics: myPrometheusRecorder,

    // Leave Codec as default (MsgPack) unless you need human-readable Redis values.
}
```

---

## Build tags

| Tag | Purpose |
|-----|---------|
| `dev` | Activates development helpers (build info, extra logging). Required for `make test` and many test files in this repo. |
| `integration` | Enables integration tests that spin up real Postgres + Redis via Testcontainers. Not run by default. |

```bash
go test -tags dev ./...         # unit tests
go test -tags integration ./... # integration tests (requires Docker)
make all                         # lint + vet + test + coverage report
```

The binary's version string is injected via `-ldflags`:

```bash
go build -tags dev \
  -ldflags "-X 'github.com/AndrewDonelson/strata.BuildDate=2026.03.01-1200' \
            -X 'github.com/AndrewDonelson/strata.BuildEnv=prod'" .
```

---

## Key internal constants (relevant for white-box tests)

| Identifier | Value | Meaning |
|-----------|-------|---------|
| `l1WriteBufSize` | `512` | Capacity of the L1 async-write channel |
| default L1 shards | `256` | Number of independent L1 shards |
| `defaultInvalidationChannel` | `"strata:invalidate"` | Redis pub/sub channel for cross-node invalidation |

---

## L4 — Distributed Sync Layer (integrated)

### Key design principle

L4 is **not standalone**. It is baked into the Strata write path. When `Config.L4.Enabled = true` and a schema has `L4.Enabled = true`, every successful L3 write automatically publishes a signed, hash-chained record to the distributed ledger — no extra code needed.

Write order with L4 active:
```
L3 write (confirmed) → L4 Publish → L2 write → L1 write
```

For `WriteBehind`: L4 publishes inside `flushDirty`, **after** the L3 write is confirmed.  
For `WriteThrough` / `WriteThroughL1Async`: L4 publishes synchronously in the same `Set` call.

L4 errors are **logged and swallowed** — they never cause `Set`/`Delete` to return an error. L3 is always the source of truth.

---

### Enabling L4 globally — `strata.L4Config`

Add to `strata.Config`:

```go
strata.Config{
    PostgresDSN: "...",
    RedisAddr:   "...",
    L4: strata.L4Config{
        Enabled:        true,
        Mode:           "ledger",   // "peer" or "ledger"
        Port:           7743,       // default
        DataDir:        "/var/lib/myapp/l4",
        SyncInterval:   30 * time.Second,
        MaxPeers:       50,
        Quorum:         3,
        BootstrapPeers: []string{"10.0.0.2:7743"},
        NodeKeyPath:    "/var/lib/myapp/l4/node.key",
    },
}
```

Fields & defaults:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Enabled` | `bool` | `false` | Must be `true` for L4 to operate |
| `Mode` | `string` | `"peer"` | `"peer"` = in-memory; `"ledger"` = BoltDB-backed |
| `Port` | `int` | `7743` | TCP listen port |
| `DataDir` | `string` | `/var/lib/strata/l4` | BoltDB directory (ledger mode) |
| `SyncInterval` | `time.Duration` | `30s` | Gossip frequency |
| `MaxPeers` | `int` | `50` | Max simultaneous peer connections |
| `Quorum` | `int` | `3` | Confirmations for `pending → confirmed` |
| `BootstrapPeers` | `[]string` | nil | `"host:port"` peers to dial on startup |
| `DNSSeed` | `string` | `""` | DNS seed hostname for peer discovery |
| `NodeKeyPath` | `string` | `""` | Path to persist Ed25519 node key |

---

### Per-schema opt-in — `strata.L4Policy`

```go
strata.Schema{
    Name:  "leaderboard",
    Model: &LeaderboardEntry{},
    L4: strata.L4Policy{
        Enabled:     true,    // opt this schema into L4 sync
        AppID:       "bounty-hunters", // L4 namespace; defaults to schema Name
        SyncDeletes: true,    // Delete() → L4 Revoke(); false = L4 record is not revoked
    },
}
```

Schemas with `L4.Enabled = false` (the default) are completely unaffected even when global L4 is on.

---

### Write path integration points (source: `router.go`, `sync.go`)

| Method | File | Where L4 fires |
|--------|------|----------------|
| `routerSetWriteThrough` | router.go | After `writeToL3` succeeds |
| `routerSetL1Async` | router.go | After `writeToL3` succeeds |
| `routerSetWriteBehind` | sync.go | Inside `flushDirty`, after `writeToL3` succeeds |
| `routerDelete` | router.go | After `l3.DeleteByID` succeeds (only if `SyncDeletes: true`) |

Helper methods defined in `strata.go` (never need direct import of `internal/l4` in `router.go`):
- `(*DataStore).syncToL4(ctx, cs, id, value)` — calls `l4layer.Publish`
- `(*DataStore).revokeFromL4(ctx, cs, id)` — calls `l4layer.Revoke`
- `structToL4Payload(value any)` — JSON-round-trips value to `map[string]interface{}`

---

### `DataStore` fields added

```go
l4layer  l4pkg.L4Layer  // nil when L4 disabled
l4nodeID string         // Ed25519 pub-key hex of this node (cached at init)
```

`l4layer` is initialised in `NewDataStore` if `cfg.L4.Enabled`, then shut down in `Close`.

---

### L4 record payload

The `payload` published to L4 is the JSON representation of the struct value, as `map[string]interface{}`. `omit_cache` and `strata:"-"` fields are included in JSON by default unless the struct uses `json:"-"` tags.

Each L4 record gets:
- `UUID` = the schema record ID (primary key)
- `AppID` = `L4Policy.AppID` (or schema name)
- `NodeID` = `ds.l4nodeID` (this node's Ed25519 public key hex)
- `Hash` = SHA-256 over `prevHash|appID|uuid|payload|timestamp`
- `PrevHash` = hash of previous record in the AppID chain (`"genesis"` for first)

---

### Direct L4 access (import `internal/l4`)

For building audit UIs, ledger readers, or cross-node queries, import the L4 package:

```go
import "github.com/AndrewDonelson/strata/internal/l4"
```

| Function | Returns | Use |
|----------|---------|-----|
| `l4.New(cfg)` | `L4Layer, error` | Create standalone layer |
| `l4.NewWithComponents(cfg, signer, store, transport)` | `L4Layer, error` | DI constructor |
| `l4.NewSigner()` | `L4Signer, error` | Fresh Ed25519 keypair |
| `l4.NewSignerFromKey(privKey)` | `L4Signer` | From existing key |
| `l4.NewMemStore()` | `L4Store` | In-memory store |
| `l4.NewBoltStore(dir)` | `L4Store, error` | BoltDB store |
| `l4.NewTCPTransport(nodeID, maxPeers, handler)` | `L4Transport` | Production transport |
| `l4.NewMemTransport(nodeID, maxPeers, hub, handler)` | `L4Transport` | Test transport |
| `l4.NewMemTransportHub()` | `*MemTransportHub` | In-process wire |
| `l4.NewAPIServer(layer, store, transport)` | `*APIServer` | HTTP API |

`L4Layer` interface:

```go
type L4Layer interface {
    Publish(appID, nodeID string, payload map[string]interface{}) (L4Record, error)
    Query(appID, recordID string) (L4Record, error)
    Revoke(appID, recordID string) error
    Subscribe(appID string, handler RecordHandler) error
    Unsubscribe(appID string) error
    Status() L4Status
    PeerCount() int
    Shutdown() error
}
```

---

### L4 errors

All compatible with `errors.Is`. Never propagated by `Set`/`Delete`.

```go
l4.ErrL4Disabled        // Config.L4.Enabled = false
l4.ErrInvalidL4Mode     // Mode not "peer"/"ledger"
l4.ErrInvalidQuorum     // Quorum < 1
l4.ErrAlreadyPublished  // duplicate UUID+AppID
l4.ErrAlreadyRevoked    // already revoked
l4.ErrNotFound          // not found locally
l4.ErrInvalidSignature  // Ed25519 failure
l4.ErrChainBreak        // hash chain violated
l4.ErrNoPeers           // no peers connected
l4.ErrQuorumNotMet      // still pending
l4.ErrStoreUnavailable  // store unavailable (peer mode)
```

---

### Testing patterns

#### Unit tests — disable L4 (zero overhead)

```go
ds, _ := strata.NewDataStore(strata.Config{
    // L4 not set → defaults to Enabled: false
})
_ = ds.Register(strata.Schema{Name: "players", Model: &Player{}})
// L4 never fires
```

#### Integration tests — peer mode, in-memory transport

```go
ds, _ := strata.NewDataStore(strata.Config{
    PostgresDSN: testDSN,
    L4: strata.L4Config{
        Enabled: true,
        Mode:    "peer",
        Quorum:  1,
    },
})
_ = ds.Register(strata.Schema{
    Name:  "leaderboard",
    Model: &LeaderboardEntry{},
    L4:    strata.L4Policy{Enabled: true},
})
_ = ds.Migrate(ctx)
defer ds.Close()

_ = ds.Set(ctx, "leaderboard", "vox", &LeaderboardEntry{XP: 100})
// L4 record is automatically published
```

#### Direct L4 store read (ledger mode)

```go
store, _ := l4.NewBoltStore(t.TempDir())
records, _ := store.Latest("bounty-hunters", 50)
```

#### HTTP API server

```go
srv := l4.NewAPIServer(layer, store, transport)
ln, _ := net.Listen("tcp", "127.0.0.1:0")
go srv.ListenOnListener(ln)
defer srv.Shutdown(context.Background())
resp, _ := http.Get("http://" + ln.Addr().String() + "/peers")
```

---

### Anti-patterns

| ❌ Don't | ✅ Do instead |
|----------|--------------|
| Enable L4 per-schema but forget `Config.L4.Enabled = true` | Always set global `Config.L4.Enabled = true` first |
| Expect `Set` to fail when L4 is down | L4 errors are logged only; `Set` still succeeds |
| Read L4 records to make critical decisions during `Set` | L4 is eventual — use L3 for authoritative reads |
| Use `Mode: "ledger"` without a `DataDir` | Default DataDir is `/var/lib/strata/l4`; override in prod |
| Set `Quorum > clusterSize` | Records stay `pending` forever; use `Quorum: 1` in tests |

---

## Vector Search (L3 Semantic Layer)

Strata supports `pgvector`-powered approximate nearest-neighbour search.
Vectors live **only in L3** (never in L1/L2); the rest of the record is cached normally.

### What you need

| Requirement | Notes |
|---|---|
| `github.com/pgvector/pgvector-go v0.3.0` | already in `go.mod` |
| Postgres with `pgvector` extension | `CREATE EXTENSION IF NOT EXISTS vector;` |
| An `EmbeddingProvider` implementation | Built-ins: `NewOllamaProvider`, `NewOpenAIProvider` |

### Defining a vector schema

```go
import pgvector "github.com/pgvector/pgvector-go"

type FAQ struct {
    ID         string          `strata:"primary_key"`
    Question   string
    CustomerID string
    Embedding  pgvector.Vector `strata:"vector"` // L3-only; never in L1/L2
}
```

Register with an optional HNSW or IVFFlat index:

```go
prov := strata.NewOllamaProvider("http://localhost:11434", "nomic-embed-text")

ds, _ := strata.NewDataStore(strata.Config{
    EmbeddingProvider: prov,
    // … L2, L3 DSN etc.
})

_ = ds.Register(strata.Schema{
    Name:  "faq",
    Model: &FAQ{},
    Indexes: []strata.Index{
        {
            Fields:         []string{"embedding"},
            Type:           strata.IndexHNSW,
            M:              16,
            EfConstruction: 64,
            DistanceFunc:   "cosine",
        },
    },
})
_ = ds.Migrate(ctx)
```

### VectorSearch

```go
results, err := ds.VectorSearch(ctx, "faq", "How do I refund?", 10, map[string]any{
    "customer_id": "cust-42",
})
for _, r := range results {
    faq := r.Value.(*FAQ)
    fmt.Printf("score=%.4f  %s\n", r.Score, faq.Question)
}
```

`VectorSearch` embeds the query text, executes an ANN query in Postgres, and
warms L2/L1 caches on the results (vector fields stripped before cache storage).

### ReEmbed (model migration)

```go
// After switching to a new embedding model:
err := ds.ReEmbed(ctx, "faq", "Question")
```

`ReEmbed` is resumable — it persists progress in `strata_schema_meta` and picks
up where it left off if interrupted.

### Built-in providers

```go
// Ollama (dimension auto-detected on first call)
prov := strata.NewOllamaProvider("http://cqai:11434", "nomic-embed-text")

// OpenAI (dimension read from well-known map or detected on first call)
prov := strata.NewOpenAIProvider(os.Getenv("OPENAI_API_KEY"), "text-embedding-3-small")
```

Both implement:
```go
type EmbeddingProvider interface {
    Embed(ctx context.Context, text string) (pgvector.Vector, error)
    Dimensions() int   // called once at Register time
    ModelID() string   // stored in strata_schema_meta; change triggers ErrEmbeddingModelChanged
}
```

### Index types

| Constant | Postgres index | Use when |
|---|---|---|
| `strata.IndexDefault` (empty) | btree / sequential scan | small tables, testing |
| `strata.IndexIVFFlat` | `ivfflat` | fast build, moderate recall; set `Lists` |
| `strata.IndexHNSW` | `hnsw` | high recall, slower build; set `M`, `EfConstruction` |
| `strata.IndexTrigram` | `gin (gin_trgm_ops)` | full-text similarity on plain columns |

Distance functions: `"cosine"` (default), `"l2"`, `"ip"` (inner product).

### Vector errors

| Error | Meaning |
|---|---|
| `ErrNoVectorField` | Schema has no `strata:"vector"` field |
| `ErrPgvectorExtensionMissing` | `CREATE EXTENSION vector` not run in Postgres |
| `ErrVectorDimensionMismatch` | Provider returned 0 or wrong dimension |
| `ErrInvalidIndexType` | IVFFlat/HNSW index targets a non-vector field |
| `ErrTopKInvalid` | `topK < 1` |
| `ErrNoEmbeddingProvider` | `Config.EmbeddingProvider` is nil |
| `ErrEmbeddingModelChanged` | Model ID in `strata_schema_meta` differs from provider |
| `ErrReEmbedAlreadyRunning` | Another `ReEmbed` is in progress |
| `ErrReEmbedTextFieldMissing` | `textFieldName` not found in model struct |
| `ErrInvalidTagForType` | `strata:"vector"` on a non-`pgvector.Vector` field |
| `ErrEmptyVectorQuery` | Blank/whitespace query string to `VectorSearch` |
| `ErrNilContext` | nil `ctx` passed to `VectorSearch` or `ReEmbed` |

### Anti-patterns

| ❌ Don't | ✅ Do instead |
|---|---|
| Hardcode dimension numbers (`vector(768)`) | Let the provider supply dimensions; set `Config.EmbeddingProvider` |
| Expect vector fields in L1/L2 cache | Vector fields are always stripped; fetch from L3 or VectorSearch |
| Run `ReEmbed` while keeping same schema name + changed model | Strata detects model change via `strata_schema_meta`; it returns `ErrEmbeddingModelChanged` so you know to call `ReEmbed` |
| Use IVFFlat/HNSW on a non-vector column | Only `pgvector.Vector` fields support ANN index types |
| Call `VectorSearch` without calling `Migrate` first | `Migrate` creates the `vector(N)` column and `strata_schema_meta`; vector queries will fail without it |
| Forget `CREATE EXTENSION IF NOT EXISTS vector` | `Migrate` checks for the extension and returns `ErrPgvectorExtensionMissing` |

---

*Skill maintained alongside the Strata source at `/home/andrew/Development/Golang/Strata/SKILL.md`.*

---
> Source: [AndrewDonelson/Strata](https://github.com/AndrewDonelson/Strata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

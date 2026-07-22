## netsy

> - **Always use `make build`** to build — never use `go build` directly. `make build` injects version info via linker flags that `go build` misses.

# Netsy — Agent Development Guide

### Important

- **Always use `make build`** to build — never use `go build` directly. `make build` injects version info via linker flags that `go build` misses.
- **Always use `make proto`** to regenerate protobuf files — never run `protoc` directly.

## Build & Test Commands

- **Setup**: `make setup` — verify required tools are installed and enable git hooks
- **Build**: `make build` — builds netsy binary with version info
- **Test**: `make test` — run all tests with race detector
- **Format**: `make fmt` — format Go source files
- **Lint**: `make lint` — run golangci-lint
- **Precommit**: `make precommit` — run pre-commit checks (fmt, lint, test)
- **Clean**: `make clean` — remove `bin/` directory
- **Protobuf**: `make proto` — generates Go files from proto files in `./proto`
- **Test Package**: `go test -v -race ./internal/peerapi/` — run specific package tests

## Dev Environment Commands

- **Start**: `make start` — starts fake S3 + Air/Netsy via overmind (auto-generates dev certs)
- **Stop**: `make stop` — stop overmind, remove `temp/`, remove `.overmind.sock`
- **Restart**: `make restart` — restart all Netsy instances managed by overmind
- **Status**: `make status` — show overmind process status
- **Tail**: `make tail` — tail all dev log files
- **Attach**: `make attach` — attach to overmind tmux session

### Dev Logs

Log files are written to `temp/logs/`. To view in real-time:

```
overmind echo netsy
overmind echo s3
```

## Documentation

- `docs/design/` — internal design documents (observability, architecture)
- `docs/deployment/` — deployment guides (config reference, TLS certificates, systemd)
- `docs/development.md` — local development setup
- `docs/motivation.md` — project motivation

## Testing

- **Fakes over mocks**: use hand-written fakes composed from existing interfaces (`storage.ObjectStorage`, `localdb.Database`). No mock libraries.
- **Frameworks**: stdlib `testing` only. Table-driven where multiple sub-cases exist. Direct assertions with `t.Fatal`/`t.Fatalf`.
- **Test placement**: tests live in the package they exercise (white-box). Name test files after their subject (`quorum_test.go`, `preflight_test.go`), not after a category.
- **Slow tests**: don't preemptively add skip guards. If a test turns out genuinely slow (and can't be sped up), add `testing.Short()` + `t.Skip("skipping slow test")` to just that test, then `go test -short` will skip it. Don't use build tags for test filtering.
- **SQLite in tests**: use `localdb.New(filepath.Join(t.TempDir(), "test.sqlite3"))` for real SQLite — don't mock the database.
- **Object storage in tests**: use `storage.NewMemoryStore()` for in-memory fake object storage.
- **Test helpers**: use `t.Helper()` on all helper functions. Use `t.Cleanup()` for teardown.

## Code Style

- **File headers**: Include copyright and SPDX license header on all files
- **Package docs**: use `doc.go` for package-level documentation
- **Imports**: stdlib → third-party → local (`github.com/netsy-dev/netsy/*`)
- **Naming**: PascalCase types/methods, camelCase variables, lowercase packages
- **Errors**: named returns `(result Type, err error)`, early returns, `fmt.Errorf()` wrapping
- **Comments**: function name first, describe purpose
- **Context**: always pass `context.Context` as the first parameter
- **Logging**: always use `log/slog` — no other logging libraries
- **Dependencies**: never add new Go module dependencies without explicit confirmation
- **TODOs**: leave existing TODOs unless they are directly related to the current task
- **Style**: straightforward, idiomatic Go

## Directory Structure

```
cmd/                     # Executable entry points
examples/                # Example configuration files
internal/                # Private application packages
proto/                   # Protobuf schema definitions
scripts/                 # Dev and CI helper scripts
docs/
├── deployment/          # Deployment guides (config, TLS, systemd)
├── design/              # Internal design documents
├── development.md       # Local development setup
└── motivation.md        # Project motivation
```

## Go Package Overview

```
cmd/
├── dev-s3/              # Local fake S3 server for development
├── netsy/               # Main netsy binary entry point
└── read-netsy-file/     # CLI tool to read .netsy data files

internal/
├── bootstrap/           # Node startup loading and backfill from object storage
├── buildvars/           # Build-time version info injected via linker flags
├── clientapi/           # etcd-compatible gRPC Client API (Range, Txn, Watch, Lease, MemberList, Status)
├── cmd/                 # Root CLI command and server lifecycle wiring
├── commonapi/           # Shared request handling logic used by Client and Peer APIs
├── config/              # Environment variable and JSONC config file loading and validation
├── datafile/            # .netsy data file format reader/writer with CRC validation
├── datastore/           # Object storage chunk and snapshot management
├── discovery/           # Node registration and member ID mapping in object storage
├── elector/             # s3lect leader election, Elector gRPC service, Primary election
├── healthserver/        # HTTP /health endpoint driven by Health State
├── heartbeat/           # Outbound heartbeat sender to Elector and Primary
├── localdb/             # SQLite database operations, schema, and migrations
├── mtls/                # Mutual TLS configuration, certificate validation, and identity extraction
├── node/                # Node gRPC service (GetNodeState, PushClusterState, etc.)
├── nodestate/           # Runtime state tracking (Health, Elector, Primary, Cluster State)
├── peerclient/          # Outbound gRPC connection manager for peer Nodes
├── primary/             # Primary gRPC service, transaction processing, quorum, chunk buffer
├── proto/               # Generated protobuf Go types for Elector, Primary, Node services
├── replication/         # Replica-side Follow stream client and self-degradation
├── snapshot/            # Periodic snapshot creation worker
├── storage/             # Object storage abstraction layer (S3, GCS, in-memory)
└── watch/               # Watch subscription manager and event distribution
```

---
> Source: [netsy-dev/netsy](https://github.com/netsy-dev/netsy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->

## keyop-messenger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

Phases 1–16 complete.
See `DESIGN.md` for the full architecture and `IMPLEMENTATION.md` for the phase plan.

## Build / Test / Install Commands

```bash
go test -race ./...                                              # run all unit tests
go test -race -v ./...                                          # verbose
go test -race -tags integration -timeout 60s ./...              # run integration tests
go test -run='^$' -bench=. -benchmem -benchtime=3s ./...        # run benchmarks
go build ./...                                                  # build library + CLI
go install ./cmd/keyop-messenger                                # install CLI binary
go mod tidy                                                     # sync dependencies
golangci-lint run ./...                                         # lint (requires golangci-lint)
```

Or via `make`:

```bash
make test               # unit tests with race detector
make test-integration   # integration tests (build tag: integration)
make bench              # benchmarks
make lint               # golangci-lint
make build              # verify compilation
```

## Environment Variables

| Variable | Description |
|---|---|
| `KEYOP_MESSENGER_DATA_DIR` | Override the data directory at runtime (alternative to config file) |
| `KEYOP_MESSENGER_CONFIG` | Path to a YAML config file (overrides built-in defaults) |

## Architecture Summary

Keyop Messenger is a Go pub-sub library with:
- Append-only `.jsonl` segment files as the durable message store (one directory per channel)
- Per-subscriber byte-offset tracking for at-least-once delivery
- Federation via mTLS WebSocket connections between instances
- Star topology: clients connect to hubs; clients declare which channels they want via `subscribe` in the handshake; hubs enforce per-client channel allowlists (`allow_channels`) and compute the effective set as `subscribe ∩ allowlist`
- Hub `peer_hubs[].forward` is the allowlist for peer hub subscriptions (not an auto-push list)
- Audit log (`{data_dir}/audit/audit.jsonl`) for all cross-hub forwarding events
- `keyop-messenger keygen ca` / `keygen instance` CLI for certificate generation

## Key Packages

| Package | Role |
|---|---|
| `github.com/wu/keyop-messenger` | Public API (`Messenger`, `New`, `Publish`, `Subscribe`, `Close`) |
| `internal/storage` | Writer goroutine, subscriber goroutine, offset files, compaction |
| `internal/federation` | Hub, Client, PeerSender, PeerReceiver, policy hot-reload |
| `internal/envelope` | Envelope struct, marshal/unmarshal, payload registry |
| `internal/tlsutil` | TLS config builder, cert generation, expiry checks |
| `internal/audit` | Audit writer with rotation |
| `internal/dedup` | LRU seen-ID deduplication |
| `cmd/keyop-messenger` | CLI entry point (`keygen ca`, `keygen instance`, `version`) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## karat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

**Prerequisite:** `CGO_ENABLED=1` is required (confluent-kafka-go uses librdkafka).

```bash
# Build
go build -o karat

# Run
go run main.go

# Test (always use -race)
go test -race ./...
go test -race ./pkg/util -run TestFunctionName

# Format (must run before commits)
golangci-lint fmt -v

# Lint
golangci-lint run
go vet ./...
```

Max line length is 120 characters (enforced by golines in `.golangci.yml`).

## Architecture

Karat is an event-driven Terminal UI for Apache Kafka built with [tview](https://github.com/rivo/tview).

**Package responsibilities:**
- `pkg/ui/` — all TUI components; the bulk of the application (~21 files)
- `pkg/client/` — Kafka `AdminClient` wrapper, holds per-cluster client instances
- `pkg/config/` — YAML config loading, style config, path resolution
- `pkg/connect/` — Kafka Connect REST API client
- `pkg/schemaregistry/` — Schema Registry client
- `pkg/shell/` — CLI template execution (kcat, kafka-console-consumer)
- `pkg/util/` — formatting helpers, modal utilities

**Event-driven UI pattern:**
Each resource type (topics, consumer groups, subjects, connectors, etc.) has a dedicated typed event channel. Operations are published via `Publish(ch, eventType, payload)` and handled in goroutines started by `app.go`. Handlers call `QueueUpdateDraw` to update the UI without blocking. Events carry a `Force` flag to bypass the in-memory cache (5-min TTL).

**Application startup (`app.go::Run()`):**
1. Load config and color theme
2. Spin up all event handler goroutines with a shared context
3. Initialize tview layout (`Layout`, `PagesRegistry`)
4. Auto-select cluster/registry/connect from config
5. Set up key bindings and run the tview application

**Page navigation:** `PagesRegistry` tracks open pages. Users navigate via `:` (resource menu), opened-pages modal, and keyboard shortcuts. Pages are cached and reused.

**Client lifecycle:** Kafka, Schema Registry, and Connect clients are lazily initialized per cluster and stored in maps on `App`.

## Code Style

Every Go file must start with this copyright header:
```go
// Copyright (c) Sergey Petrovsky
// This source code is licensed under the MIT license found in the
// LICENSE file in the root directory of this source tree.
```

Import grouping (blank-line separated):
1. Standard library
2. External dependencies
3. Internal packages (`github.com/uraniumdawn/karat/pkg/...`)

All exported functions, types, and methods must have godoc comments starting with the entity name. Pass `context.Context` as the first parameter for any blocking operation.

---
> Source: [uraniumdawn/karat](https://github.com/uraniumdawn/karat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->

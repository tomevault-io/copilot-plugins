## tndrl

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What is Tndrl?

A control plane for distributed AI agents built on the [A2A protocol](https://a2a-protocol.org/).

Tndrl provides a unified interface for orchestrating AI agents running across multiple machines, containers, and environments. It uses a **peer-to-peer** model where any node can both serve requests and connect to other nodes.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         tndrl node                          │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐ │
│  │   A2A       │    │   Control   │    │      LLM        │ │
│  │   Server    │    │   Server    │    │    Provider     │ │
│  └─────────────┘    └─────────────┘    └─────────────────┘ │
│         │                  │                    │          │
│         └──────────────────┼────────────────────┘          │
│                            │                               │
│                    ┌───────┴───────┐                       │
│                    │  QUIC/mTLS    │                       │
│                    └───────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Other Nodes   │
                    │   (peers)       │
                    └─────────────────┘
```

| Component | Purpose | Details |
|-----------|---------|---------|
| **tndrl node** | Unified daemon/client | Runs `tndrl serve` or connects as client |
| **A2A Server** | Agent communication | [A2A protocol](https://a2a-protocol.org/) for prompts, tasks |
| **Control Server** | Lifecycle management | Ping, status, shutdown |
| **LLM Provider** | Language model backend | echo, ollama (pluggable) |

## Key Design Decisions

- **Single binary** — one `tndrl` binary for all roles (daemon and client)
- **Peer-to-peer** — any node can both serve and connect to other nodes
- **A2A protocol alignment** — agent communication follows the [A2A spec](https://a2a-protocol.org/)
- **Go** — single binary, easy cross-compilation, good concurrency
- **QUIC transport** — modern, multiplexed, connection migration
- **mTLS everywhere** — mutual TLS with built-in CA, SPIFFE-compatible identities
- **Config-driven** — unified CLI/env/file configuration with versioned config schema

## Code Structure

```
tndrl/
├── cmd/tndrl/           # unified CLI (serve, ping, prompt, etc.)
│   ├── main.go          # entry point, Kong setup
│   ├── cli.go           # CLI struct, config types, loading
│   ├── serve.go         # daemon mode
│   ├── client.go        # shared connection logic
│   └── *.go             # subcommands (ping, status, prompt, etc.)
├── examples/            # example config files
│   ├── echo.yaml        # testing with echo provider
│   └── ollama.yaml      # production with Ollama
├── gen/go/tndrl/v1/     # generated protobuf/gRPC code
├── pkg/
│   ├── pki/             # CA and certificate management
│   ├── transport/quic/  # multiplexed QUIC transport
│   ├── control/         # ControlService implementation
│   ├── a2aexec/         # A2A executor (agent logic)
│   └── llm/             # LLM provider abstraction
├── proto/tndrl/v1/      # protobuf definitions
├── buf.yaml             # buf configuration
└── buf.gen.yaml         # code generation config
```

## Quickstart

```bash
# Terminal 1: Start a node as a daemon
tndrl serve -c examples/echo.yaml

# Terminal 2: Interact with the node (using named peer from config)
tndrl prompt -c examples/echo.yaml local "Hello!"
tndrl status -c examples/echo.yaml local
tndrl discover -c examples/echo.yaml local
```

PKI files are stored in `~/.tndrl/pki/`. See [pkg/pki/README.md](./pkg/pki/README.md) for details.

## Documentation

- [Configuration Reference](./docs/configuration.md) — config file format and options
- [CLI Reference](./docs/cli.md) — all commands and flags
- [Protobuf & buf](./docs/protobuf.md) — schema definitions, code generation
- [PKI & Security](./pkg/pki/README.md) — certificate management

### Design Documents

- [A2A Alignment](./docs/design/a2a-alignment.md) — adopting A2A protocol
- [Protocol](./docs/design/protocol.md) — wire format, message types

## Status

See **[docs/PROJECT.md](./docs/PROJECT.md)** for current progress and next steps.

**Current**: LLM integration complete (Ollama). Next: tool execution framework.

## Development Environment

Prefer running commands inside `toolbox` when available. The host system may not have development tools like `go` installed.

```bash
# Enter toolbox before running go commands
toolbox run go test ./...
toolbox run go build ./cmd/...
```

## Testing

Tests use [goleak](https://github.com/uber-go/goleak) for goroutine leak detection and race detection is enabled by default.

```bash
# Run all tests with race detection (default)
make test

# Verbose output
make test-verbose

# With coverage report
make test-cover

# Unit tests only (skip integration)
make test-unit

# Integration tests only
make test-integration
```

Each test package has a `TestMain` that runs goleak verification after all tests complete. If a test leaks goroutines, it will fail.

## Git Workflow

We use a fork-based workflow:
- `upstream` = main repo (tndrl/tndrl)
- `origin` = your fork

Branch protection requires PRs for all changes to main. Always use fetch/rebase:

```bash
# Update main from upstream
git checkout main
git fetch upstream main
git rebase upstream/main

# Create a feature branch
git checkout -b feature/my-feature

# ... make changes, commit ...

# Push to your fork and create PR against upstream
git push -u origin feature/my-feature
gh pr create --repo tndrl/tndrl

# After PR is merged, sync main and clean up
git checkout main
git fetch upstream main
git rebase upstream/main
git push origin main
git branch -d feature/my-feature
git push origin --delete feature/my-feature

# Update current branch with latest main (without switching)
git fetch upstream main
git rebase upstream/main

# Move uncommitted work to a new branch after PR merge
git fetch upstream main
git rebase upstream/main
git checkout -b new-branch-name

# If upstream has diverged (e.g., squash merge, force push)
git checkout -b backup/my-feature  # backup first
git checkout my-feature
git fetch upstream
git reset --hard upstream/main
```

**Never use `git pull` or `git merge`** — always fetch then rebase (or reset --hard only when upstream has diverged).

## When Working Here

1. Read the component README for context before making changes
2. Capture design decisions in documentation as they're made
3. Keep docs minimal — prefer pointers over duplication
4. **Always update [docs/PROJECT.md](./docs/PROJECT.md)** when completing work:
   - Move completed items from "In Progress" or "Next Steps" to "Completed"
   - Add new tasks discovered during implementation to "Next Steps"
   - Update "Current Objective" if focus has shifted
   - Keep the tracking document accurate — it's the source of truth for project state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tndrl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tndrl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

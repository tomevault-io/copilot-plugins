## cloudctl

> This document provides essential context and technical guidelines for AI agents and developers working on the `cloudctl` project.

# Agent Development Guidelines for cloudctl

This document provides essential context and technical guidelines for AI agents and developers working on the `cloudctl` project.

## Project Overview
`cloudctl` is a Go-based CLI tool (Go 1.25) designed to manage Kubernetes cluster access via the Greenhouse platform. Its primary purpose is to aggregate and sync kubeconfigs from a central Greenhouse management cluster to a user's local configuration.

### Core Stack
- **Language**: Go 1.25+
- **CLI Framework**: [Cobra](github.com/spf13/cobra)
- **Kubernetes Client**: `k8s.io/client-go` and `sigs.k8s.io/controller-runtime/pkg/client`
- **Testing**: Standard Go tests + [Gomega](github.com/onsi/gomega) for E2E assertions.

## Development Workflows

### Build & Run
- **Build binary**: `make build` (outputs to `bin/cloudctl`)
- **Run without build**: `make run ARGS="version"`
- **Install to GOBIN**: `make install`

### Testing
- **Unit Tests**: `make test`
- **E2E Tests**: `make e2e`
    - *Note*: E2E tests require `k3d`. The `Makefile` manages cluster lifecycle.
    - E2E tests are located in `/e2e` and use the `e2e` build tag.

### Modern Go Idioms (Go 1.25)
Always use modern Go features:
- **Looping**: `for i := range n` instead of `for i := 0; i < n; i++`.
- **Slices/Maps**: Use `slices` and `maps` packages (e.g., `slices.Contains`, `maps.Clone`).
- **Standard Library**: Use `max()`, `min()`, and `clear()`.
- **Iterators**: Use `maps.Keys(m)` or `maps.Values(m)` with `slices.Collect`.
- **Error Handling**: Use `errors.Is` and `errors.As`.
- **JSON tags**: Use `omitzero` instead of `omitempty` for `time.Time`, `time.Duration`, structs, slices, and maps.

## Project Structure & Core Logic

### Key Directories
- `/cmd`: CLI command implementations (using Cobra).
- `/e2e`: End-to-end tests and `k3d` lifecycle scripts.
- `/hack`: Utility scripts and internal tools.

### Command Implementation (`/cmd`)
- `root.go`: Defines the root command and global helpers like `configWithContext`.
- `sync.go`: Contains the core logic for merging kubeconfigs.
    - It fetches `v1alpha1.ClusterKubeconfig` resources from Greenhouse.
    - It merges clusters, contexts, and auth infos while preserving user modifications to unmanaged entries.
    - It handles `oidc-login` (kubelogin) configuration.
- `cluster-version.go`: Implements Kubernetes version detection (unauthenticated fallback to authenticated).

### Kubeconfig Management
When modifying `sync.go`, ensure:
1. **Deduplication**: AuthInfos (users) are merged if they represent the same credentials (checked via `authInfoEqual`).
2. **Context Prefixing**: Remote clusters and contexts are typically prefixed to avoid collisions.
3. **Immutability**: Do not overwrite manual/unmanaged entries in the user's kubeconfig unless they overlap with managed entries.

## Commit Guidelines
- **DCO**: Ensure all commits have signed-off (`git commit -s`).
- **Style**: Use `type(scope): short description` style (e.g., `feat(controller): add dry-run support`).
- **Note**: Do NOT add `Co-authored-by` trailers unless explicitly requested.

## Agent Checklist (Definition of Done)
- [ ] Code style? Followed Modern Go idioms and ran `make fmt`.
- [ ] Testing? Added/updated unit tests and verified with `make test`.
- [ ] E2E? Verified with `make e2e` if affecting `sync` or `cluster-version`.
- [ ] Commits? All commits are signed-off (`-s`).

---
> Source: [cloudoperators/cloudctl](https://github.com/cloudoperators/cloudctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->

## waspctl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language conventions

When writing prose (Portuguese), keep technical terms in English as they are referenced in the project — e.g., "Hierarchical Platform Topology", "Auth Service", "tenant", "platform cluster". Do not translate these.

## Project Overview

`waspctl` is a Go CLI tool for managing a multi-tenant Kubernetes platform (WASP). It provisions and manages a hierarchical topology of platform clusters and customer clusters across AWS regions, with a single global entry point (`wasp.silvios.me`).

## CLI Conventions

Commands follow the pattern `waspctl <resource> <verb> [flags]`. Command groups: `config`, `provider`, `dns`, `registry`, `database`, `instance`, `customer`, `tenant`, `auth`.

**Global flags** (all commands): `-o/--output table|json|yaml` (default: `table`), `-y/--yes` (skip confirmation), `--config <path>` (default: `~/.wasp/config.yaml`).

**`config` command** follows the `git config` convention — it manages the config file directly (e.g., `waspctl config --set provider aws`, `waspctl config --get provider`, `waspctl config --list`). Do not conflate with `provider` subcommands.

**`customer` commands** always require `--instance <platform-name>` — this is intentional for safety and clarity, so the user always explicitly declares which platform-cluster the customer-cluster belongs to.

**Confirmation prompt** — commands that create cloud resources ask for confirmation of AWS account + region unless `-y/--yes` is passed.

**Always update this file** at the end of sessions where architectural decisions, CLI conventions, or design patterns are established.

## Architecture

### Tenant Routing

Single domain (`wasp.silvios.me`) → Global Accelerator → regional ALB → Auth Service resolves tenant from email domain (DynamoDB lookup) → sets JWT/cookie → ALB listener rules route to the correct customer cluster gateway.

### Platform Topology

```
wasp.silvios.me
└── Global Accelerator
    ├── platform-cluster-use1 (us-east-1)
    │   ├── customer1-use1
    │   └── customer2-use1
    └── platform-cluster-brs1 (sa-east-1)
        ├── customer3-brs1
        └── customer4-brs1
```

### Key AWS Components

- **Route 53**: resolves `wasp.silvios.me` to Global Accelerator
- **Global Accelerator**: anycast, geo-routes to nearest platform cluster
- **ALB**: cookie/header-based routing to customer cluster target groups
- **DynamoDB Global Table**: tenant registry (`email_domain → tenant_gateway`)
- **EKS**: platform clusters and customer clusters
- **Cognito**: OIDC/OAuth with Lambda Triggers to inject tenant into JWT

### In-Cluster Stack

Istio (service mesh + gateway), ArgoCD (GitOps), external-dns, cert-manager, Crossplane (undercloud resource management via k3d locally).

## Development Phases

1. Single cluster + simple auth service
2. Platform cluster separated from customer clusters
3. Regional platform clusters + Global Accelerator + DynamoDB Global Table

## Tests

All commands should have unit tests for core logic and integration tests when it makes sense if the command interact with AWS, should mock AWS services (using tools like `localstack` or `moto`) to validate end-to-end flows without incurring cloud costs during development.

After each change, the test suite should be run to ensure no regressions. For critical flows (like tenant routing), consider adding end-to-end tests that deploy a test cluster and validate routing behavior.

### Running tests

Use `make test` to compile and run the test suite. `go test` via wrapper is killed by the sandbox — the Makefile compiles the test binary directly to avoid this.

```bash
make clean build test
```

`go build ./...` and `go vet ./...` work normally and should always pass before committing.

### Build

Use `make build` to compile and install the binary to `~/bin/waspctl`. Ensure `~/bin` is in your `$PATH`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smsilva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

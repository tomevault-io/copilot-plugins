## cupi-cli

> Go CLI tool (`cupi`) for Cisco Unity Connection (CUC). Wraps:

# cupi — Cisco Unity Connection CLI

## Overview

Go CLI tool (`cupi`) for Cisco Unity Connection (CUC). Wraps:
- **CUPI** — REST/JSON provisioning (`/vmrest/`) — users, distlists, handlers, COS, templates, schedules
- **PAWS** — Platform admin SOAP (`/platform-services/services/`) — cluster status, DRS backup
- **AST** — System health XML (`/ast/Astisapi.dll`) — disk, heartbeat, alerts, perfmon
- **DIME** — Log collection (`/logcollectionservice/services/DimeGetFileService`) — log file downloads

## Key Paths

- Config: `~/.cupi/config.json` (servers map, not `clusters`)
- Keyring service: `cupi`
- Module: `github.com/Cloverhound/cupi`
- Binary: `cupi`
- Env vars: `CUPI_DEBUG`, `CUPI_DRY_RUN`

## Credential Types

- `cupi` — CUPI REST (default, analogous to `axl` in cucm-cli)
- `application` — Application-level APIs
- `platform` — OS admin for PAWS

## Development

```bash
go build -o cupi .        # Build (Version reports "dev")
./cupi --help             # Verify
./cupi version            # Print version
./cupi upgrade --check    # Check for newer release on GitHub
bash install-local.sh     # Install binary (embeds git tag as version) + skill
go run ./tools/scraper/   # Run API doc scraper
go test ./tests/ -run TestCLI  # Run CLI validation tests (no live server needed)

# Build with explicit version:
go build -ldflags "-X github.com/Cloverhound/cupi-cli/cmd.Version=v1.2.3" -o cupi .
```

## Architecture

- `internal/appconfig/` — `ServerConfig` (not `ClusterConfig`), `defaultServer`, `servers`
- `internal/auth/` — `CredTypeCUPI="cupi"`, keyring service `"cupi"`
- `internal/client/rest.go` — JSON REST: `Request/Get/Post/Put/Delete`
- `internal/client/ast.go` — VOS-common AST only (disk, tftp, heartbeat, alerts, perfmon)
- `internal/client/paws.go` — PAWS SOAP (identical to cucm-cli, CUPI_DEBUG)
- `internal/client/dime.go` — DIME log download (identical to cucm-cli, CUPI_DEBUG)
- `tools/scraper/` — Phase 1 HTML scraper for CUPI API docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cloverhound) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

## homelab-mcp-server

> MCP server exposing homelab gateway capabilities as AI tools for the Homelab AI Platform.

# homelab-mcp-server

MCP server exposing homelab gateway capabilities as AI tools for the Homelab AI Platform.
Part of the [homelab-ai](https://github.com/lobo235/homelab-ai) platform.

## Module

`github.com/lobo235/homelab-mcp-server`

## Quick Start

```bash
cp .env.example .env
# Fill in required values
go run ./cmd/server
```

## Build, Test, Run

> Go is installed at `~/bin/go/bin/go` (also on `$PATH` via `.bashrc`).

```bash
# Build
make build

# Run tests
make test

# Run tests with verbose output
go test -v ./...

# Run linter
make lint

# Coverage report (opens in browser)
make cover

# Run the server (requires .env or env vars)
make run

# Build binary
go build -o homelab-mcp-server ./cmd/server

# Run integration tests (requires integration build tag)
go test -tags integration ./internal/discovery/
```

## Project Layout

```
homelab-mcp-server/
├── Dockerfile
├── Makefile
├── go.mod / go.sum
├── .env.example              # dev template — never commit real values
├── .gitignore
├── .golangci.yml             # strict linter config
├── .githooks/pre-commit      # runs lint + tests; activate with `make hooks`
├── CLAUDE.md                 # this file
├── README.md
├── CHANGELOG.md
├── cmd/
│   └── server/
│       └── main.go           # entry point — MCP stdio server
└── internal/
    ├── config/
    │   ├── config.go          # ENV var loading & validation
    │   └── config_test.go
    ├── clients/
    │   ├── nomad/             # nomad-gateway HTTP client
    │   ├── adguard/           # adguard-home-gateway HTTP client
    │   ├── cloudflare/        # cloudflare-gateway HTTP client
    │   ├── filesystem/        # filesystem-gateway HTTP client
    │   ├── minecraft/         # minecraft-gateway RCON HTTP client
    │   ├── curseforge/        # curseforge-gateway HTTP client
    │   ├── modrinth/          # Modrinth public API client
    │   ├── ftb/               # FTB public API client (api.modpacks.ch)
    │   └── vault/             # vault-gateway HTTP client
    ├── discovery/             # Modpack discovery pipeline (7-stage async)
    ├── tools/
    │   ├── atomic/            # Layer 1 — single gateway call tools
    │   ├── orchestration/     # Layer 2 — multi-step with rollback
    │   └── highlevel/         # Layer 3 — user intent fulfillment
    ├── validation/            # Job spec pre-flight checks
    ├── prompts/               # MCP prompt resources
    ├── resilience/            # Retry policy, circuit breaker
    ├── speccache/             # Nomad job spec cache
    ├── itzgcache/             # itzg docs cache
    └── modpackkb/             # Modpack deployment knowledge base
```

## Configuration

All config via ENV vars. Loaded from `.env` in development (via `godotenv`; missing file silently ignored). In production, secrets are injected by the chatbot's Nomad task environment.

| Var | Required | Default | Purpose |
|-----|----------|---------|---------|
| `NOMAD_GATEWAY_URL` | yes | — | Base URL of nomad-gateway |
| `NOMAD_GATEWAY_KEY` | yes | — | API key for nomad-gateway |
| `ADGUARD_GATEWAY_URL` | yes | — | Base URL of adguard-home-gateway |
| `ADGUARD_GATEWAY_KEY` | yes | — | API key for adguard-home-gateway |
| `CF_GATEWAY_URL` | yes | — | Base URL of cloudflare-gateway |
| `CF_GATEWAY_KEY` | yes | — | API key for cloudflare-gateway |
| `MINECRAFT_GATEWAY_URL` | yes | — | Base URL of minecraft-gateway (RCON only) |
| `MINECRAFT_GATEWAY_KEY` | yes | — | API key for minecraft-gateway |
| `FILESYSTEM_GATEWAY_URL` | yes | — | Base URL of filesystem-gateway |
| `FILESYSTEM_GATEWAY_KEY` | yes | — | API key for filesystem-gateway |
| `CURSEFORGE_GATEWAY_URL` | yes | — | Base URL of curseforge-gateway |
| `CURSEFORGE_GATEWAY_KEY` | yes | — | API key for curseforge-gateway |
| `VAULT_GATEWAY_URL` | yes | — | Base URL of vault-gateway |
| `VAULT_GATEWAY_KEY` | yes | — | API key for vault-gateway |
| `LOG_LEVEL` | no | `info` | Verbosity: `debug`, `info`, `warn`, `error` |
| `NOMAD_DEFAULT_DATACENTER` | yes | — | Default Nomad datacenter for job generation |
| `NOMAD_DEFAULT_NODE_POOL` | no | `default` | Default node pool for MC jobs |
| `NFS_BASE_PATH` | yes | — | NFS base path for Minecraft server volumes |
| `VOLUME_ALLOWLIST` | no | — | Additional allowed volume mount prefixes (comma-separated) |
| `MC_PUBLIC_DOMAIN` | yes | — | Public domain for MC server CNAMEs |
| `MC_PUBLIC_IP` | no | — | Public IP for informational context (not used in operations) |
| `CF_ZONE_NAME` | yes | — | Cloudflare zone name |
| `ARTIFACT_ALLOWLIST` | no | — | Additional artifact source domains (comma-separated) |
| `DATA_DIR` | no | `/data` | Directory for spec cache, itzg docs, and state |
| `ITZG_DOCS_REFRESH_INTERVAL` | no | `24h` | How often to refresh itzg docs cache |
| `ANTHROPIC_API_KEY` | no | — | Anthropic API key for web search enrichment in discovery |
| `DISCOVERY_TEMP_DIR` | no | `/tmp/modpack-discovery` | Temp directory for modpack downloads during discovery |

## Architecture

```
cmd/server/main.go               — entry point, wires deps, starts MCP stdio server
internal/config/config.go         — ENV-based config with validation
internal/clients/nomad/           — nomad-gateway HTTP client wrapper
internal/clients/adguard/         — adguard-home-gateway HTTP client wrapper
internal/clients/cloudflare/      — cloudflare-gateway HTTP client wrapper
internal/clients/filesystem/      — filesystem-gateway HTTP client (file ops, backups, downloads)
internal/clients/minecraft/       — minecraft-gateway HTTP client (RCON only)
internal/clients/curseforge/      — curseforge-gateway HTTP client wrapper
internal/clients/vault/           — vault-gateway HTTP client wrapper
internal/tools/atomic/            — Layer 1 MCP tools (single gateway call each)
internal/tools/orchestration/     — Layer 2 MCP tools (multi-step with rollback)
internal/tools/highlevel/         — Layer 3 MCP tools (user intent fulfillment)
internal/validation/              — Job spec pre-flight validation
internal/prompts/                 — MCP prompt resources (homelab_context, etc.)
internal/resilience/              — Retry policy, circuit breaker, health checks
internal/speccache/               — Nomad job spec cache (auto-seeded on startup)
internal/itzgcache/               — itzg/docker-minecraft-server docs cache
internal/modpackkb/               — Modpack deployment knowledge base (JSON files on disk)
internal/discovery/               — Modpack discovery pipeline (resolve, download, extract, analyze, enrich, finalize)
internal/clients/modrinth/        — Modrinth public API HTTP client
internal/clients/ftb/             — FTB public API HTTP client (api.modpacks.ch)
```

## MCP Tools

All tools are registered with mcp-go v0.45.0 and served via stdio transport.

### Layer 1 — Atomic Tools

| Tool | Gateway | Description |
|------|---------|-------------|
| `list_running_jobs` | nomad | List all running Nomad jobs |
| `get_job_spec` | nomad | Get original HCL spec for a job |
| `get_job_status` | nomad | Get job status + allocations |
| `get_job_logs` | nomad | Get allocation logs (requires `task`, optional `log_type`) |
| `submit_nomad_job` | nomad | Submit HCL job spec (with pre-flight validation) |
| `stop_nomad_job` | nomad | Stop/purge a job |
| `restart_nomad_allocation` | nomad | Restart an allocation |
| `watch_job_health` | nomad | Check job health status |
| `create_cloudflare_record` | cloudflare | Create a DNS record |
| `delete_cloudflare_record` | cloudflare | Delete a DNS record |
| `create_local_dns_rewrite` | adguard | Create AdGuard DNS rewrite |
| `delete_local_dns_rewrite` | adguard | Delete AdGuard DNS rewrite |
| `create_server_secret` | vault | Create Minecraft server secrets |
| `delete_server_secret` | vault | Delete Minecraft server secrets |
| `create_workload_secret` | vault | Create generic workload secret (admin only) |
| `delete_workload_secret` | vault | Delete generic workload secret (admin only) |
| `init_server_directory` | minecraft | Initialize server NFS directory |
| `delete_server_directory` | minecraft | Delete server NFS directory |
| `execute_rcon_command` | minecraft | Send RCON command to server |
| `list_backups` | minecraft | List server backups |
| `create_backup` | minecraft | Create server backup |
| `download_to_server` | minecraft | Download file (modpack, mod, config) to server directory |
| `list_archive_contents` | minecraft | List files inside a zip/tar archive on server |
| `list_server_files` | minecraft | List files and directories on server filesystem |
| `read_server_file` | minecraft | Read a file from server filesystem |
| `write_server_file` | minecraft | Write content to a file on server filesystem |
| `move_server_file` | minecraft | Move or rename a file/directory on server filesystem |
| `delete_server_file` | minecraft | Delete a file or directory from server filesystem |
| `fetch_artifact` | — | Fetch trusted artifact content (scripts, configs) by URL |
| `search_itzg_docs` | — | Search itzg/docker-minecraft-server documentation for a keyword |
| `get_itzg_doc` | — | Read a specific itzg/docker-minecraft-server documentation page |
| `search_modpacks` | curseforge | Search CurseForge for modpacks by name |
| `search_mods` | curseforge | Search CurseForge for mods by name |
| `validate_modpack` | curseforge | Validate a CurseForge modpack |
| `get_modpack_files` | curseforge | List modpack file versions |
| `get_modpack_file` | curseforge | Get specific modpack file by file ID |
| `validate_mod` | curseforge | Validate a CurseForge mod |
| `get_mod_file` | curseforge | Get specific mod file by file ID |
| `get_modpack_knowledge` | — | Look up modpack deployment knowledge by name/slug/CF ID |
| `save_modpack_knowledge` | — | Save/update modpack deployment knowledge (admin-only) |
| `list_modpack_knowledge` | — | List all known modpacks in the knowledge base |
| `delete_modpack_knowledge` | — | Delete a modpack from the knowledge base (admin-only) |
| `trigger_modpack_discovery` | — | Start async discovery pipeline for an unknown modpack |
| `get_discovery_state` | — | Check status/progress of a running discovery pipeline |

### Layer 2 — Orchestration Tools

| Tool | Description |
|------|-------------|
| `provision_minecraft_server` | Init dir -> create secret -> submit job -> create DNS -> wait health |
| `destroy_minecraft_server` | Async: stop job -> delete DNS -> delete secret -> (optionally) delete dir. Returns immediately. |
| `get_destroy_status` | Check progress of an async server destruction |
| `provision_nomad_workload` | Submit job -> create AdGuard DNS rewrite (admin only) |
| `destroy_nomad_workload` | Stop job -> delete AdGuard DNS rewrite (admin only) |
| `add_mod_to_server` | Validate mod, resolve required dependencies, download mod + deps to server mods/ dir |

### Layer 3 — High-Level Task Tools

| Tool | Description |
|------|-------------|
| `create_minecraft_server` | Select reference spec, generate HCL, provision |
| `destroy_minecraft_server_by_name` | Destroy by server name |
| `upgrade_minecraft_server` | Backup -> update spec -> resubmit |
| `get_minecraft_server_status` | Aggregate job state + allocation health |
| `deploy_generic_workload` | Generate HCL from description, provision (admin only) |

## Testing Approach

Tests live alongside their packages in `*_test.go` files.

Key patterns:
- Gateway client tests use `httptest.NewServer` to mock gateway HTTP APIs
- Config tests cover all required fields, defaults, and validation
- Validation tests cover all pre-flight check rules
- Table-driven tests for input validation
- Both success and error paths tested

## Naming Convention — Minecraft Servers

Minecraft server names follow a split naming convention:

| Concern | Convention | Example |
|---------|-----------|---------|
| Nomad job ID | `mc-{name}` | `mc-atm9` |
| Vault secret path | `mc-{name}` | `kv/nomad/default/mc-atm9` |
| RCON (via minecraft-gateway) | `mc-{name}` | `mc-atm9` (gateway looks up Nomad allocations) |
| NFS directory | `{name}` | `/minecraft/atm9/` |
| DNS hostname | `{name}.{domain}` | `atm9.example.com` |
| Backups (via minecraft-gateway) | `{name}` | `atm9` |

Tool layers use `validation.MCServerDir(jobID)` to strip the `mc-` prefix for NFS, backup, and DNS operations. RCON passes the full job ID because the minecraft-gateway uses it to query Nomad for the allocation's dynamic RCON port.

## Coding Conventions

- Uses `github.com/mark3labs/mcp-go` v0.45.0 with **stdio transport only** (HTTP/SSE has race condition)
- No external router, ORM, or framework — minimal dependency footprint
- All gateway calls include `Authorization: Bearer <KEY>` and `X-Trace-ID` headers
- All upstream errors wrapped with `fmt.Errorf("context: %w", err)`
- Structured JSON logging via `log/slog` to stderr (stdout reserved for MCP stdio)
- Version logged on startup
- Never log secret values (API keys, passwords, tokens)
- **UI-schema sync:** When data schemas change (e.g. `ModpackKnowledge`, `VersionKnowledge`, `ExtractedData`, MCP tool response shapes), flag that the chatbot's admin panel and frontend may need corresponding updates. Schema changes here often require form/display updates in `homelab-chatbot/internal/frontend/index.html`.

## Security Rules

> **Claude must enforce all rules below on every commit and push without exception.**

1. **Never commit secrets:** No `.env`, tokens, API keys, passwords, or credentials of any kind.
2. **Never commit infrastructure identifiers:** No real hostnames, IP addresses, datacenter names, node pool names, Consul service names, Vault paths with real values, Traefik routing rules with real domains, or any value that reveals homelab architecture. Use generic placeholders (`dc1`, `default`, `example.com`, `your-node-pool`, `your-service`).
3. **Unknown files:** If `git status` shows a file Claude didn't create, ask the operator before staging it.
4. **Pre-commit checks (must all pass before committing):**
   - `go test ./...` — all tests must pass
   - `golangci-lint run` — no lint errors
5. **Docs accuracy:** Review all changed `.md` files before committing — documentation must reflect the current state of the code in the same commit.
6. **Version bump:** Before any `git commit`, review the changes and determine the appropriate SemVer bump (MAJOR/MINOR/PATCH). Present the rationale and proposed new version to the operator and wait for confirmation before tagging or referencing the new version.
7. **Push confirmation:** Before any `git push`, show the operator a summary of what will be pushed (commits, branch, remote) and wait for explicit confirmation.
8. **Commit messages:** Must not contain real hostnames, IPs, or infrastructure identifiers.

## Versioning & Releases

SemVer (`MAJOR.MINOR.PATCH`). Git tags are the source of truth.

```bash
git tag v1.2.3 && git push origin v1.2.3
```

This triggers the Docker workflow which publishes:
- `ghcr.io/lobo235/homelab-mcp-server:v1.2.3`
- `ghcr.io/lobo235/homelab-mcp-server:v1.2`
- `ghcr.io/lobo235/homelab-mcp-server:latest`
- `ghcr.io/lobo235/homelab-mcp-server:<short-sha>`

Version is embedded at build time: `-ldflags "-X main.version=v1.2.3"` — defaults to `"dev"` for local builds. Logged on startup.

## Docker

```bash
# Build (version defaults to "dev")
docker build -t homelab-mcp-server .

# Build with explicit version
docker build --build-arg VERSION=v1.2.3 -t homelab-mcp-server .
```

Multi-stage build: `golang:1.26-alpine` -> `alpine:3.21`. Statically compiled (`CGO_ENABLED=0`).

## Known Limitations

- **stdio transport only:** HTTP/SSE transport has an active race condition in mcp-go (issue #763). Pinned to v0.45.0.
- **No standalone Nomad job:** Runs exclusively as a subprocess of the chatbot, not as its own Nomad service.
- **Gateway dependency:** If a gateway is unreachable, its tools are marked unavailable but the MCP server continues running.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lobo235)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lobo235)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

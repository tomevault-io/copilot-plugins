## homelab-horizon

> **Generated:** 2026-01-04

# HOMELAB-HORIZON KNOWLEDGE BASE

**Generated:** 2026-01-04
**Commit:** 3328a95
**Branch:** main

## OVERVIEW

Self-contained homelab management: WireGuard VPN, split-horizon DNS (dnsmasq + Route53/Name.com), HAProxy reverse proxy with Let's Encrypt SSL, service health monitoring. Single Go binary, runs on Ubuntu/Debian.

## STRUCTURE

```
homelab-horizon/
├── cmd/homelab-horizon/     # Entry point, CLI flags, systemd install
├── internal/
│   ├── server/              # HTTP handlers, HTML templates (6k lines) ← AGENTS.md
│   ├── config/              # JSON config loading, validation, derivation
│   ├── wireguard/           # WG config parsing, key generation, interface mgmt
│   ├── haproxy/             # Config generation, reload, backend management
│   ├── letsencrypt/         # ACME client, cert scheduling, renewal
│   ├── dnsmasq/             # Internal DNS config, hosts file management
│   ├── dns/                 # DNS provider abstraction (Route53, Name.com)
│   ├── route53/             # AWS Route53 API, public IP detection, IPv6 check
│   ├── acme/                # ACME provider factory for lego
│   ├── monitor/             # Health checks, ntfy notifications
│   ├── system/              # FileSystem/CommandRunner interfaces (for testing)
│   └── qr/                  # QR code SVG generation
├── test/integration/        # Dry-run integration tests
└── docs/screenshots/        # UI screenshots
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add HTTP endpoint | `internal/server/server.go` | `setupRoutes()` method |
| Add handler | `internal/server/handlers_*.go` | Group by domain |
| Modify UI | `internal/server/templates_*.go` | Raw HTML strings, no templating engine |
| Add config field | `internal/config/config.go` | Add JSON tag, update `derive.go` |
| DNS provider | `internal/dns/` | Implement `Provider` interface |
| Test mocks | `internal/system/` | `DryRunFileSystem`, `DryRunCommandRunner` |

## CONVENTIONS

- **Templates**: Raw HTML strings in `templates_*.go`, no external files
- **Config derivation**: Services/zones derive DNS mappings, HAProxy backends, SSL domains via `derive.go`
- **JSON tags**: All config fields use `json:"snake_case"` with `omitempty`
- **Version injection**: `main.Version` set via `-ldflags` at build time
- **CSRF**: All POST forms require token from meta tag, auto-injected by JS

## ANTI-PATTERNS (THIS PROJECT)

- **DEPRECATED handlers**: `handlers_haproxy.go` and `handlers_setup.go` have deprecated Route53/HAProxy direct management → now auto-derived from services
- **No pkg/**: Everything is internal, no public API
- **No Dockerfile**: Designed for bare-metal systemd deployment
- **Templates are massive**: `templates_admin.go` is 1k+ lines → accept it, don't refactor

## COMMANDS

```bash
make                    # Build for current platform
make build-all          # Cross-compile: amd64, arm64, armv7
make test               # Run all tests
make test-unit          # internal/ tests only
make test-integration   # test/integration/ only
make check              # go vet + go fmt
sudo ./homelab-horizon  # Requires root for WG/ports 80/443
./homelab-horizon -dry-run    # Preview changes without applying
./homelab-horizon -check      # Interactive system check
./homelab-horizon -version    # Show version
```

## TESTING

- **Dry-run mode**: `system.DryRunFileSystem` + `system.DryRunCommandRunner` mock all I/O
- **Test files**: `*_test.go` alongside implementation
- **Integration**: `test/integration/dry_run_test.go` tests full server lifecycle

## NOTES

- Requires sudo: WireGuard interface, ports 80/443, iptables
- Config search: `./config.json`, `./homelab-horizon.json`, `/etc/homelab-horizon/config.json`
- Admin token: Generated on first run, written to `{config}.token`
- IPv6 check: `route53.CheckIPv6()` tests connectivity via api6.ipify.org

---
> Source: [IodeSystems/homelab-horizon](https://github.com/IodeSystems/homelab-horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->

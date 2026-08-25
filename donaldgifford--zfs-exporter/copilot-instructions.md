## zfs-exporter

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

A Prometheus exporter for ZFS hosts, written in Go 1.25.4. Ships as a complete
monitoring package: exporter binary, Grafana dashboards, and Prometheus alert
rules. Targets Debian/Linux ZFS hosts (also builds for macOS). Fills gaps left
by pdf/zfs_exporter and node_exporter's ZFS collector by combining ZFS
pool/dataset metrics, share discovery (NFS/SMB), and service health into one
deployable unit. See `docs/V1_PLAN.md` for full implementation plan.

## Build & Development Commands

```bash
make build              # Build binary to build/bin/zfs_exporter
make test               # Run all tests with race detector (go test -v -race ./...)
make test-pkg PKG=./pkg/zfs  # Test a specific package
make test-coverage      # Tests with coverage report (coverage.out)
make test-report        # Generate and open HTML coverage report
make lint               # Run golangci-lint
make lint-fix           # Auto-fix lint issues
make fmt                # Format with gofmt + goimports (local prefix: github.com/donaldgifford)
make check              # Pre-commit: lint + test
make ci                 # Full CI pipeline: lint + test + build
make run-local          # Build and run the exporter
make dashboards         # Regenerate Grafana dashboard JSON and Prometheus rules
make lint-dashboards    # Validate dashboard PromQL, metrics, and panel structure
# Dashboard generator tests (separate module):
# cd tools/dashgen && go test -race ./...
make release-check      # Validate goreleaser config
make release-local      # Test goreleaser without publishing
```

Tool versions are managed via `mise.toml`. Install tools with `mise install`.

## Architecture

This is a Prometheus exporter following the standard collector pattern. Package
layout:

- **`cmd/zfs_exporter/`** - Entry point: flag parsing, HTTP server setup,
  graceful shutdown
- **`collector/`** - Prometheus collector implementing `Describe` and `Collect`
  methods
- **`config/`** - Configuration (kingpin CLI flags + env var overrides) and
  sentinel validation errors
- **`exporter/`** - HTTP handlers (landing page)
- **`pkg/zfs/`** - Public ZFS client package. Executes `zpool` and `zfs` CLI
  commands on the local host and parses their output (including
  `sharenfs`/`sharesmb` share properties and `zpool status` scan state for
  resilver/scrub detection). No HTTP client (this exporter runs directly on the
  ZFS host). Uses a `Runner` function type for command execution, enabling test
  injection of fixture data without interface mocking.
- **`pkg/host/`** - Host service checker. Uses `systemctl is-active` to check
  systemd unit states for a configurable list of services (default: ZFS, NFS,
  SMB, iSCSI). Reuses the `Runner` type from `pkg/zfs/`.
- **`tools/dashgen/`** - Dashboard code generator (separate Go module). Uses the
  Grafana Foundation SDK to produce dashboard JSON from a Go config struct. Run
  via `make dashboards` or `cd tools/dashgen && go generate .`. Config in
  `config.go`, panel builders in `panels/`, dashboard assemblers in
  `dashboards/`.
- **`contrib/grafana/`** - Generated Grafana dashboards: Status (quick-glance
  stat panels), Details (all graphs/tables), Combined (status panels +
  expandable drill-down rows). Files: `zfs-status.json`, `zfs-details.json`,
  `zfs-combined.json`. Regenerate with `make dashboards`.
- **`contrib/prometheus/`** - Pre-built Prometheus alert rules and recording
  rules. Alerts cover pool health, drive failure/rebuild, capacity thresholds,
  service down, share/service mismatches, and anomaly detection (dataset growth
  outside normal range, pool fill prediction). Recording rules pre-compute 1-day
  and 7-day baselines for anomaly detection.
- **`deploy/deb/`** - (Planned) Debian packaging: systemd service,
  postinstall/preremove scripts, lintian overrides

### Key Patterns (from GUIDE.md)

**Collector**: Use `MustNewConstMetric` for all metrics (never direct
instrumentation). Define metric descriptors once in the constructor with
`prometheus.NewDesc` and `prometheus.BuildFQName(namespace, subsystem, name)`.
Create fresh metrics on each scrape in `Collect()`. Always emit `up` and
`scrape_duration_seconds`.

**Error handling in Collect**: Required endpoint failures set `up=0` and return
early. Optional endpoint failures log a warning and continue with fallback
values.

**Client**: Lives in `pkg/zfs/` as a public package. Context as first parameter.
Wrap errors with `%w`. Executes `zpool`/`zfs` commands with parseable flags
(`-Hp`) and explicit column selection (`-o`). Binary paths configurable via
`--zfs.zpool-path` and `--zfs.zfs-path` flags (validated at startup).

**Testing**: Use injected `Runner` functions with fixture data for `pkg/zfs/`
tests (analogous to `httptest.Server` pattern from GUIDE.md). Use
`testutil.CollectAndCompare` for collector metric validation. Table-driven tests
for parsing and error scenarios.

**Configuration**: kingpin for CLI flags, env var overrides, sentinel errors for
validation. Precedence: defaults -> flags -> env vars. Service list configurable
via `--host.services` (default: `zfs,nfs,smb,iscsi`).

**Logging**: stdlib `slog` (no external logging dependencies).

**Main**: HTTP server with all four timeouts (ReadHeader, Read, Write, Idle).
Graceful shutdown on SIGINT/SIGTERM.

## Code Style

Follows the **Uber Go Style Guide** enforced by a strict `.golangci.yml`:

- Import ordering enforced by gci: standard library, third-party, then
  `github.com/donaldgifford` local packages
- Max cyclomatic complexity: 15, cognitive complexity: 30, function lines: 100,
  nesting: 4
- All `//nolint` directives require explanation and specific linter name
- Errors must always be checked, wrapped with `%w`, and types ending in "Error"
- HTTP requests must use context; response bodies must be closed
- Structs > 80 bytes should be passed by pointer
- Max line length: 150 (enforced by golines)

## Release & Packaging

- Releases via goreleaser: static binaries (CGO_ENABLED=0) for linux/darwin on
  amd64/arm64
- Debian packages (.deb) with systemd integration built via goreleaser nfpms
- Version info injected via ldflags: `main.Version`, `main.Commit`,
  `main.BuildDate`
- Conventional commits: `feat:`, `fix:` prefixes drive changelog and semver
  bumps
- PR labels (major/minor/patch/dont-release) control automated version bumping
- Coverage target: 60% (threshold: 40%)

## CI/CD

GitHub Actions workflows:

- **test.yml**: Runs on push to main and all PRs. Jobs: auto-labeler, lint
  (golangci-lint v2.8.0), test with coverage (Codecov), build verification
  (goreleaser snapshot), Debian package lintian validation.
- **release.yml**: On push to main, auto-bumps semver from PR labels, tags, runs
  goreleaser with GPG signing.
- **pr-labels.yml**: Requires exactly one semver label on every PR.

---
> Source: [donaldgifford/zfs_exporter](https://github.com/donaldgifford/zfs_exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->

## openclaw-boilerplate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Security-focused, fully automated deployment boilerplate for [OpenClaw](https://openclaw.ai) (open-source AI agent framework). Wraps the official OpenClaw installer with server hardening, automated backups, and security update infrastructure. Not a library — this is infrastructure-as-code (shell scripts, Docker, CI).

**Boilerplate version is aligned to OpenClaw releases: currently v2026.3.13.**

When adapting to a new OpenClaw version:
1. Read the upstream changelog: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md
2. Check the config reference: https://docs.openclaw.ai/gateway/configuration-reference
3. Check security docs: https://docs.openclaw.ai/gateway/security
4. Bump `.release-please-manifest.json` to match the upstream version
5. Update this file's version number

## Commands

### Lint
```bash
shellcheck -S warning setup.sh openclaw-update.sh openclaw-restore.sh test/test-setup.sh
```

### Test (Docker smoke test)
```bash
docker build -f test/Dockerfile.test -t openclaw-test .
docker run --rm --privileged openclaw-test
```

### Validate config templates
```bash
python3 -c "import json; json.load(open('openclaw.json.example'))"
python3 -c "import json; json.load(open('.release-please-manifest.json'))"
docker compose config --quiet  # validates docker-compose.yml
```

## Architecture

The boilerplate has three layers:

1. **Bootstrap (`setup.sh`)** — 21-step idempotent script run as root on a fresh Linux server. Creates the `openclaw` user, hardens SSH/firewall, installs Docker/Tailscale/Node.js/OpenClaw (all checksum-verified), generates `openclaw.json`, and starts the daemon. Contains an inline fallback copy of `openclaw-update.sh` — **both copies must stay in sync**.

2. **Runtime config (`openclaw.json.example`, `docker-compose.yml`, `.env.example`)** — Templates with hardened defaults. `setup.sh` generates the actual `openclaw.json` dynamically (the example file is documentation only). The Docker path is an alternative to the systemd daemon path.

3. **Maintenance (`openclaw-update.sh`, `openclaw-restore.sh`)** — Root-owned scripts installed to `/usr/local/bin/`. The update script runs weekly via cron with backup, version logging, and post-update security audit.

### Key design decisions

- Gateway always binds to `127.0.0.1:18789`, never `0.0.0.0`. Remote access is via Tailscale or SSH tunnel only.
- `auth: "none"` was permanently removed upstream in v2026.1.29. Config always uses `auth.mode: "token"`.
- `tools.profile` is explicitly set to `"full"` to prevent the v2026.3.2 bug that defaulted to `"messaging"`.
- `plugins.security.autoLoadWorkspace` is `false` to block supply chain attacks from cloned repos.
- `tools.exec.ask` is `"always"` — every shell command requires user approval.
- `browser.enabled` is `false` — browser automation off by default (SSRF risk).
- `session.maintenance.mode` is `"enforce"` with 30d prune to prevent unbounded disk growth.
- `controlUi.dangerouslyDisableDeviceAuth` is explicitly `false`.
- Channel credentials flow through `sanitize_for_json()` to prevent JSON injection.

## Commit Conventions

Uses [Conventional Commits](https://www.conventionalcommits.org/) with release-please:
- `fix:` = patch, `feat:` = minor, `feat!:` = major
- `ci:`, `chore:`, `docs:`, `test:` = no release
- `security:` = patch (appears under "Security" in changelog)

## CI Pipeline

All GitHub Actions pinned to immutable SHA (not tags). Dependabot updates SHAs weekly.

| Workflow | Tool | Purpose |
|---|---|---|
| `lint.yml` | ShellCheck | Shell script quality |
| `security.yml` | TruffleHog + Gitleaks + Trivy | Secrets and misconfig scanning |
| `test.yml` | Docker | Smoke test `setup.sh` in simulated VPS |
| `release.yml` | release-please | Automated versioning gated on all CI passing |

## Shell Script Rules

- `umask 077` at top of every script
- `set -euo pipefail` always
- Separate `local` from command substitution (SC2155)
- `trap` for temp file cleanup
- `printf '%s'` over `echo` for variables
- File permissions: 600 for secrets/config, 700 for dirs/scripts

## Docker Hardening Invariants

When editing `docker-compose.yml`, these must be preserved:
- `read_only: true` + explicit `tmpfs` mounts
- `cap_drop: ALL`, no `cap_add`
- `no-new-privileges:true`
- Memory/CPU/PID limits and log rotation
- Health check present
- Port binding to `127.0.0.1` only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcelbaklouti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## vps-bootstrap

> > **Read `docs/ARCHITECTURE.md` for the complete architecture reference.**

# CLAUDE.md — Project Instructions for Claude Code

> **Read `docs/ARCHITECTURE.md` for the complete architecture reference.**
> It contains exact patterns, templates, checklists, and the decision tree for implementing services.

## Project Overview

VPS bootstrap system: Terraform + Bash scripts that deploy a hardened Debian 12 VPS with WireGuard VPN, Docker, Traefik, and optional services (Gitea, n8n, gogcli, whoami, mkdocs).

## Critical Rules

1. **Everything is a Docker container** on `vpn_net` (10.20.0.0/24) with static IP
2. **NEVER use `ports:` directive** — all access via Traefik (web) or `docker exec` (CLI)
3. **Traefik uses file provider** — patch `dynamic.yml` via embedded Python3, NOT Docker labels
4. **Credentials:** Terraform → `.env` → container (never hardcode)
5. **Every container:** `no-new-privileges`, `cap_drop: ALL`, joined to `vpn_net`
6. **Idempotency:** `file_matches` + `install_content` for all file writes
7. **DRY_RUN:** Every function must support `DRY_RUN=true`

## Architecture Reference

The complete guide lives in `docs/ARCHITECTURE.md` — it contains:
- Network layout with IP allocation table
- Library API reference (logging.sh, backup.sh)
- Analysis of all 4 existing service implementations
- Step-by-step instructions for web and CLI services (10 steps each)
- Complete 200-line service script template
- Traefik patching pattern
- Decision tree for AI assistants
- Full checklist

## When Adding a Service

Touch these files in order:
1. `variables.tf` — `enable_<name>` + secret vars
2. `main.tf` — `random_password` + `env_content` entries
3. `bootstrap/apply.sh` — flag + `run_service_module` call
4. `bootstrap/services/<name>.sh` — the service script
5. `terraform.tfvars.example` — example config
6. `outputs.tf` — credentials + services outputs
7. `tests/smoke.sh` — structure + syntax tests
8. `docs/SERVICES.md` — documentation
9. `docs/README.md` 

## Next Available IPs

10.20.0.71+ (used: .10 Traefik, .20 whoami, .30-.32 Gitea, .40-.41 n8n, .50 gogcli, .60+.62 MkDocs, .70 Kuma)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tabee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

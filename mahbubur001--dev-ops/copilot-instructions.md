## dev-ops

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation-only DevOps knowledge base — no application code, no build system, no tests. All content lives under `Instructions/` as Markdown guides. The `README.md` serves as the index.

## Infrastructure Context

- **Primary server:** Hetzner (`87.99.130.89`) — Ubuntu 24, 4 vCPU / 16GB RAM
- **Secondary:** AWS EC2 (`t3.medium`, Ubuntu 24.04, 30GB)
- **Database:** PostgreSQL 18
- **Hetzner DB:** `bikribd` / user `bikribdu`
- **Scripts live on server at:** `/usr/local/bin/`

## Adding or Updating Guides

1. Create the file under `Instructions/` with a descriptive kebab-case name
2. Add a link to `README.md` under the relevant section
3. Keep headings consistent with existing guides (H1 title, H2 sections, fenced bash blocks)

## README Structure

`README.md` is the top-level index. Every new `Instructions/*.md` file must be linked there. Current sections: PostgreSQL installation, backup, remote access, and server management scripts.

## Server Management Scripts

Seven scripts are maintained at `/usr/local/bin/` on the Hetzner server:

| Script | Purpose |
|---|---|
| `pg-manage.sh` | Interactive colored-menu launcher for all scripts below |
| `pg-backup.sh` | Daily automated PostgreSQL backup with retention |
| `pg-restore.sh` | Restore from a backup file |
| `pg-create-db.sh` | Create a new database and user |
| `pg-drop-db.sh` | Drop a database (and optional role) with a forced safety backup |
| `health-check.sh` | Server resource and service health report |
| `security-check.sh` | Security posture audit |

Full script contents and setup instructions: [`Instructions/server-management-scripts.md`](Instructions/server-management-scripts.md)

## PostgreSQL Remote Access

Direct remote access is IP-allowlisted at two layers: `pg_hba.conf` (Postgres) + the Hetzner cloud firewall (network). When adding a new client IP, both must be updated and PostgreSQL restarted. SSH tunnel alternative is documented for removing direct access.

See [`Instructions/postgresql-remote-access-hetzner.md`](Instructions/postgresql-remote-access-hetzner.md)

## Backup Strategy

Backups run via cron, write to `/var/backups/postgresql/`, and rotate on a 7-day retention. S3 offload and daily/weekly/monthly tiering options are documented.

See [`Instructions/postgresql-auto-backup-guide.md`](Instructions/postgresql-auto-backup-guide.md)

---
> Source: [mahbubur001/dev-ops](https://github.com/mahbubur001/dev-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

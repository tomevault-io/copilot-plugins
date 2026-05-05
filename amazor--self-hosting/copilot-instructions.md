## homelab-context

> Homelab project context, principles, and architecture


# Homelab Project — Context & Principles

This repo is a **field manual** and **source of truth** for a self-hosted homelab: part journal, part technical guide, part Infrastructure-as-Code.

## Mission

Build a robust, scalable, automated home server hosting:
- core infra (ingress, auth, DNS)
- monitoring/observability
- media automation pipelines
- general apps
- GPU workloads (transcoding/CV)

## Guiding Principles

- **Boring Core, Flexible Workloads** — Access plane is stable and predictable; workloads can churn and be rebuilt.
- **Cattle, Not Pets** — VMs are disposable. Redeploy from a known baseline instead of snowflake fixing.
- **Decoupled Compute and Data** — Proxmox provides compute; NAS provides storage. Data survives VM rebuilds.
- **Documentation-first** — Decisions include "why" notes so future-me and readers can follow the logic.

## Tech Stack

- **Hypervisor:** Proxmox VE
- **Compute:** Beelink EQi13 (Debian Cloud-Init template, Docker host)
- **Storage:** Synology NAS
- **Workloads:** Docker Compose per-VM stacks
- **Automation:** Cloud-Init + `deploy.py` orchestrator + `BootstrapRunner` framework (per-stack `stack_config.py` modules)

## Repo Structure

- **docs/** — Journey chapters + reasoning (Chapter0, Chapter1, Chapter2, Chapter2a, Chapter2c, …)
- **proxmox/** — Template automation: `scripts/`, `snippets/` (Cloud-Init)
- **scripts/** — Shared Python framework: `homelab_common.py` (helpers), `homelab_bootstrap.py` (`BootstrapRunner`), `homelab_logging.py` (`StepTracker`), `setup_env.py`
- **docker_compose/** — Per-VM stacks: `core/`, `monitoring/`, `media/`, `accelerated/` (each with `compose.yml`, `.env.example`, `bootstrap.py`, `stack_config.py`, and optional `scripts/` subdirectory)
- **docker_compose/common/** — Shared compose overlays (e.g. `compose.observability.yml` symlinked into each stack)
- **deploy.py** — Top-level orchestrator (validates env, runs bootstrap, creates symlinks/shell helpers, runs compose up)

## Architecture Rules

- **Only one VM is public** — Router forwards only 80/443 to `core`. Everything else is reachable via reverse proxy or Tailscale.
- **VM boundaries = storage boundaries** — Each VM mounts only what it needs; mounts are scoped to subfolders/exports. `core` stays minimal and typically mounts nothing.
- **Template stays boring** — Role-specific setup lives in per-VM bootstrap scripts, not in the golden image.

## Planned additions (post–current implementation)

Apps/services to add once the current implementation is done:

- **Fail2ban** — Host/access-plane protection (e.g. rate limiting whoami, SSH, Caddy logs); likely on `core` or a dedicated security layer.
- **PaperlessNGX** — Document management (scan, OCR, tag, search); planned for the `apps` VM.
- **qui** — Enhanced qBittorrent web UI (single binary, by autobrr team) for the `media` VM. Lower priority — adds tracker reannounce, automation rules, orphan scan, cross-seeding, OIDC (Authentik), and a reverse proxy so *arr apps don't need direct qBit credentials. Nice-to-have once cleanuperr covers the critical stalled-torrent loop.

When editing any part of this project, keep these principles and structure in mind.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

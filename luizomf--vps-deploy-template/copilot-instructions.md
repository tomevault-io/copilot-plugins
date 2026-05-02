## vps-deploy-template

> This document keeps future collaborators aligned on the goals, guardrails, and

# AGENTS

This document keeps future collaborators aligned on the goals, guardrails, and
deployment story of this repo.

## Mission

- This project serves as a **production-ready template** for students deploying
  Python applications on a VPS (specifically Hostinger KVM 2).
- It is the core artifact for a video tutorial series. The goal is to provide a
  "win-win-win" scenario: students get a robust starting point, the sponsor gets
  visibility, and the author provides supported, high-quality content.
- The repository is designed to be forked. It provides a full CI/CD pipeline
  bridging a local development environment with a hardened production server.

## Infrastructure Snapshot

- **Provider:** Hostinger KVM 2 VPS.
- **OS:** Ubuntu 24.04 LTS (with Docker pre-installed).
- **Network:** Ports 22 (SSH), 80 (HTTP), 443 (HTTPS) open.
- **User:** `<deploy-user>` (primary administrative user, used for deployment).
- **Directories:**
  - Project root: `/dockerlabs`
  - Persistent data: Managed via `data_vol` container and bind mounts.

## Architecture & Services

The stack is composed of Docker containers orchestrated via `compose.yaml`:

1.  **`dockerlabs` (App):**
    - Python/FastAPI application.
    - Serves the main application logic.
    - **Crucial:** Exposes `/webhook/github` endpoint to trigger deployments.
2.  **`nginx`:**
    - Reverse proxy handling SSL termination and routing.
    - Serves static assets and forwards traffic to `dockerlabs`.
3.  **`certbot`:**
    - Obtains and renews Let’s Encrypt certificates.
    - Shares volumes with NGINX for challenge response and cert storage.
4.  **`data_vol`:**
    - A lightweight container used solely for initializing and sharing volumes
      between containers and the host.

## Deployment Workflow

The deployment model has shifted from a "Push via SSH" model to a **"Signal &
Pull"** model to improve security and decoupling.

1.  **Trigger (GitHub):**
    - Developer pushes to `main`.
    - GitHub Actions builds the image and pushes it to GHCR.
    - GitHub Actions sends a POST request to the production URL
      `/webhook/github`.

2.  **Signal (App):**
    - `dockerlabs` app receives the webhook.
    - Verifies the `X-Hub-Signature-256` using `GITHUB_WEBHOOK_SECRET`.
    - If valid, it creates a timestamped file in the shared volume
      `/data/webhook_jobs`.

3.  **Execution (Watcher):**
    - A systemd service (`webhook-watcher`) running on the host checks for files
      in `/dockerlabs/data/webhook_jobs` (via `data_vol`).
    - Upon detection, it triggers `data/scripts/deploy`.

4.  **Update (Script):**
    - `data/scripts/deploy`:
      1.  Backs up volumes.
      2.  `git fetch/reset` to match `origin/main` (if in production).
      3.  `docker compose pull` & `up -d`.
      4.  Prunes old images.

## Server & Security Ground Rules

- **Access:** SSH Keys only. Password authentication is disabled.
- **Firewall:** `ufw` enabled, allowing only SSH, HTTP, and HTTPS.
- **Fail2Ban:** Configured to jail repeated failed SSH login attempts.
- **Permissions:**
  - Project directory `/dockerlabs` is owned by `<deploy-user>:<deploy-user>` with
    setgid (`chmod g+s`).
  - `webhook-watcher` runs as `<deploy-user>`.
- **Secrets:**
  - Managed via `.env` file (not committed).
  - `GITHUB_WEBHOOK_SECRET` must match between `.env` and GitHub Repository
    Secrets.

## Contribution Guidelines

- **Educational Value First:** Changes should be intelligible to students. Avoid
  over-engineering if a simpler solution exists.
- **Idempotency:** Scripts like `bootstrap` and `deploy` must be safe to run
  multiple times without breaking the state.
- **Configuration:**
  - `CURRENT_ENV` variable in `.env` controls behavior (e.g., generating dummy
    certs in `development` vs. real certs in `production`).
  - Always update `.env.example` if adding new environment variables.
- **Documentation:** Infra changes must be reflected in `README.md`. It serves
  as the script for the video tutorial.

## Quick Reference

- **Main Command:** `docker compose up -d --build` (Local dev).
- **Deploy Script:** `/dockerlabs/data/scripts/deploy`.
- **Watcher Service:** `sudo systemctl status webhook-watcher`.
- **Logs:** `sudo journalctl -u webhook-watcher -f`.

---
> Source: [luizomf/vps_deploy_template](https://github.com/luizomf/vps_deploy_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

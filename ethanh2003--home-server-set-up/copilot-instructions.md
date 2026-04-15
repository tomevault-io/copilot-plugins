## home-server-set-up

> This document serves as the primary context and instruction manual for Gemini when interacting with this project. It outlines the architecture, hardware constraints, storage topology, and workflows of the self-hosted homelab.

# Gemini Context: Self-Hosted Homelab

This document serves as the primary context and instruction manual for Gemini when interacting with this project. It outlines the architecture, hardware constraints, storage topology, and workflows of the self-hosted homelab.

## System Architecture

### Hardware Infrastructure
*   **Compute Node:** Beelink Mini S13 Pro
    *   **CPU:** Intel N150 (4 Cores/4 Threads). **Primary Bottleneck.** Optimize for low-cycle efficiency.
    *   **RAM:** 16GB DDR4.
    *   **GPU:** Intel UHD Graphics. **Requirement:** Always map `/dev/dri` for media containers (Plex/Jellyfin/Frigate) to enable hardware transcoding.
    *   **OS:** Ubuntu Server (Headless).
*   **Network:** GL.iNet GL-MT6000 (OpenWrt) + AdGuard Home.

### Storage Topology (CRITICAL)
Strict file path discipline is enforced to prevent I/O bottlenecks.

1.  **System & Application Data (Fast I/O)**
    *   **Physical:** Internal 500GB SSD.
    *   **Mount Point:** `/` (Root).
    *   **Usage:** OS, Docker Images, **Configs**, **Databases**.
    *   **Rule:** All Docker bind mounts for config/db MUST use relative paths (e.g., `./config/[service]`) within the stack directory.
    *   **Constraint:** Never put database files on USB drives.

2.  **Mass Storage (Bulk Data)**
    *   **Physical:** 14TB Recertified Enterprise HDD (USB 3.0).
    *   **Mount Point:** `/mnt/data_14tb`.
    *   **Environment Variable:** `${MEDIA_PATH}` usually points here.
    *   **Usage:** Nextcloud Data, Plex Media, Torrents.

3.  **Backup Storage (RAID 5)**
    *   **Physical:** 3x 500GB HDD (RAID 5 Array).
    *   **Mount Point:** `/mnt/aux_raid`.
    *   **Usage:** Primary Backup Target (Kopia Repositories), Database Dumps.
    *   **Redundancy:** 1-drive failure tolerance.

## Directory Structure

The project follows a **"One Folder Per Stack"** architecture.

*   **Root:** `/home/ethan/docker/`
    *   **`[stack_name]/`**: Directory containing the `docker-compose.yml` for a specific service or group (e.g., `arr-suite/`, `jellyfin/`).
        *   **`docker-compose.yml`**: Service definitions.
        *   **`config/`**: Persistent data for services (e.g., `arr-suite/config/radarr`).
        *   **`.env`**: Secrets and environment-specific variables (Git-ignored).
    *   **`manage-stacks.sh`**: Helper script to start/stop/restart/pull all stacks.
    *   **`backup_databases.sh`**: Script to dump databases (Postgres) to `.db_dumps/` for backup.
    *   **`.gitignore`**: Global ignore rules (configs, secrets, logs).

## Network Architecture

*   **External Access:** Cloudflare Tunnels (`cloudflared`) or VPN.
*   **Internal Routing:**
    *   **`proxy_net`**: A shared external Docker network. All user-facing services must connect to this network to be reachable by the reverse proxy.
    *   **VPN Routing:** The `arr-suite` uses `gluetun` as a VPN gateway. Services like `qbittorrent`, `prowlarr`, and `whisparr` use `network_mode: service:gluetun` to route traffic through the VPN.

## Operational Workflows

### 1. Stack Management
*   **Start/Update All:** `./manage-stacks.sh start` or `./manage-stacks.sh pull`
*   **Single Stack:** Navigate to the directory (e.g., `cd arr-suite`) and run standard Docker Compose commands (`docker compose up -d`).

### 2. Adding New Services
1.  **Create Directory:** `mkdir new-service && cd new-service`.
2.  **Define Compose:** Create `docker-compose.yml`.
3.  **Network:** Add to `proxy_net` if web-accessible.
4.  **Persistence:** Map volumes to `./config/[service_name]`.
5.  **Secrets:** Use `.env` for sensitive data.

### 3. Backups
*   **Database Dumps:** Run `./backup_databases.sh`. This dumps Postgres databases from containers like `immich_postgres` and `planka-postgres` to `.db_dumps/`.
*   **Kopia:** The `kopia` stack is responsible for backing up the filesystem, including `.db_dumps/`.

## Interaction Guidelines

1.  **Hardware Awareness:** If a requested service is heavy (e.g., Java game servers, LLMs), warn about the N150 CPU limitation.
2.  **Safety Check:** Ask before deleting files or stopping critical containers (especially `gluetun` or `cloudflared`).
3.  **Path Verification:** Always verify paths against the Storage Topology section.
4.  **VPN Integrity:** When modifying `arr-suite`, ensure `gluetun` connectivity is preserved for privacy-sensitive services.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ethanh2003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ethanh2003)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

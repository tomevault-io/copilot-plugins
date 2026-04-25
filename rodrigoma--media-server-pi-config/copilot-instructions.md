## media-server-pi-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains shell scripts and configuration files to set up a Raspberry Pi as a home media server. The target platform is Raspberry Pi OS (armhf). There is no build system, test suite, or package manager — this is a collection of infrastructure-as-code shell scripts meant to be run sequentially on a Pi.

## Setup Order

Scripts are numbered and must be run in order on the Pi:

1. `scripts/export_vars.sh` — Interactive script that sets required environment variables (`USERNAME`, `PASSWORD`, `SSH_PORT`, `WIFI_SSID`, `WIFI_PASS`, `TELEGRAM_CHAT_ID`, `TELEGRAM_TOKEN`, `GOOGLE_DRIVE_FOLDER_ID`, `SQLBAK_TOKEN`, `DUCKDNS_TOKEN`, `PLEX_TOKEN`, `TRAEFIK_BASIC_AUTH`) in `~/.bashrc`, `~/.profile`, and `/etc/environment`. Auto-derives `USER_ID`, `GROUP_ID`, `USER_EMAIL`, `USER_DOMAIN`, `BOT_TELEGRAM_TOKEN`, `TRANSMISSION_SERVER`, and `MEDIA_REPO_PATH`.
2. `scripts/00_0_configs_geral.sh` — General OS config (timezone, SSH, rootfs expansion). Requires `SSH_PORT` var.
3. `scripts/00_1_copy_scripts.sh` — Creates symlink `~/scripts -> ~/media-server-pi-config/scripts`, sets executable bits, registers `startup_services.sh` in crontab via `@reboot`.
4. `scripts/01_config_static_ip.sh` — Static IP config (connection drops after execution).
5. `scripts/02_install_noip.sh` — NO-IP DDNS client.
6. `scripts/03_mount_hdd.sh` — Creates `/media/14TB`, updates `/etc/fstab`, sets up home directory shortcuts (`~/digaoflix`, `~/downloads`), and schedules `wakeup_hdd.sh` every 10 minutes via crontab.
7. `scripts/04_install_samba.sh` — Samba file sharing.
8. `scripts/05_install_plex.sh` — Plex Media Server.
9. `scripts/06_0_install_docker.sh` — Docker install + interactive launcher for all container scripts.
10. `scripts/07_install_sqlbak.sh` — SqlBak backup agent (requires `SQLBAK_TOKEN`).

## Architecture

### Docker Container Stack

All containers are managed by label-based ordering:
- `sqlbak.start.first=true` — started first on boot (e.g., Watchtower)
- `sqlbak.start.first=false` — started after dependencies

Containers (each has its own `06_N_docker_run_*.sh` script):
- **Sickchill** — TV show management
- **Mylar** — Comic management
- **Transmission** — BitTorrent client (uses `USERNAME`/`PASSWORD`)
- **PyLoad** — Download manager
- **Plexdrive** — Google Drive mount (requires `GOOGLE_DRIVE_FOLDER_ID`)
- **Komga** — Comic/book server
- **WizNote** — Note-taking
- **WatchTower** — Auto-updates Docker images (`nickfedor/watchtower:armhf-latest`), runs Fridays at 17:00, sends Telegram notifications via `TELEGRAM_TOKEN`
- **Reposilite** — Maven repository
- **Apprise** — Notification service

### Startup Flow

`startup_services.sh` runs at boot via crontab. It:
1. Waits 30 seconds for system boot
2. Polls until `/media/14TB/wakeup.txt` exists (HDD is mounted)
3. Sends Telegram messages via `BOT_TELEGRAM_TOKEN`/`TELEGRAM_CHAT_ID` for status
4. Starts `sqlbak.start.first=true` containers, waits 30s, then starts remaining containers
5. Starts Plex Media Server

### SqlBak Integration

Scripts `sql-bak_before-script.sh`, `sql-bak_after-script_start-services.sh`, and `sql-bak_after-script_update-services.sh` are hooks called by SqlBak during backup jobs to stop/start services.

### Network Configuration

`configs/dhcpcd.conf.pi` configures dual static IPs for eth0 (auto-selects between 192.168.0.100 and 192.168.10.150 based on gateway ARP), and a static IP for wlan0 (192.168.10.160).

## Ansible

An `ansible/` directory exists (in progress, not in the main branch). The `.vault_pass` file is present for Ansible Vault — do not commit actual vault passwords.

## Key Conventions

- Scripts use `#!/bin/sh` (POSIX sh, not bash) except `export_vars.sh` which uses `#!/bin/bash`.
- All Docker containers use `--network host` or bind to the Pi's local network.
- The Pi user is `pi`; `$LOGNAME` is used in scripts to avoid hardcoding.
- Telegram bot notifications use `BOT_TELEGRAM_TOKEN` (format: `bot<token>`, auto-derived from `TELEGRAM_TOKEN`) for general alerts and Watchtower. `TELEGRAM_CHAT_ID` is the destination chat.
- The HDD is mounted at `/media/14TB`; older scripts reference `/media/1TB`, `/media/2TB`, `/media/4TB`, `/media/4TBII` (all commented out).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrigoma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

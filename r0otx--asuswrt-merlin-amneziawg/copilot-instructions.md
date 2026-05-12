## asuswrt-merlin-amneziawg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AmneziaWG kernel module + web UI addon for ASUS GT-AX11000 running Asuswrt-Merlin 388.x firmware. Provides DPI-obfuscated WireGuard VPN with per-device policy routing and GeoIP/GeoSite selective routing. All documentation and UI text is in Russian.

**Target:** ARM64 (aarch64), Linux kernel 4.1.51, Broadcom HND platform.

## Build

```bash
# Build kernel module + CLI tool via Docker (requires Docker Desktop, ~15GB disk)
./build.sh                    # uses existing kernel.config
./build.sh 192.168.1.1        # extracts kernel config from router via SSH first
```

Build runs entirely inside Docker (see `Dockerfile`). Outputs to `output/`:
- `amneziawg.ko` — kernel module (~200KB)
- `awg` — CLI tool (~1MB)

Cross-compiled with Broadcom aarch64 gcc 5.5 toolchain against vanilla kernel 4.1.51 sources configured with the router's `/proc/config.gz`.

## Architecture

### Build pipeline (`Dockerfile`)
Multi-stage Docker build: downloads Merlin toolchain + kernel source, applies router's kernel config, builds out-of-tree AmneziaWG kernel module and userspace tools from upstream repos. Uses `docker build --output` to export artifacts.

### Router-side components

**`addon/amneziawg.sh`** — Main backend script (runs on router). Handles:
- Interface lifecycle: `start`/`stop`/`restart` (insmod, ip link, awg setconf, iptables, ip rule)
- Config generation from Merlin's `custom_settings.txt` (key prefix: `awg_*`)
- Per-device routing policy: `vpn_all`, `vpn_geo`, `direct` via ip rules + iptables mangle marks
- GeoIP/GeoSite: downloads country CIDR lists, domain lists; populates ipset (`awg_dst`) + dnsmasq ipset rules
- Web UI addon mounting via Merlin Addons API (`am_get_webui_page`, menuTree.js bind mount)
- Service event dispatch (called from `/jffs/scripts/service-event`)

**`addon/amneziawg_page.asp`** — Web UI page (ROG-styled ASP). Communicates with backend via Merlin's `httpApi` custom settings and service events (`awgstart`, `awgstop`, `awgsaveconf`, `awgupdategeo`). Reads status from `/www/user/awg_status.htm` (JSON).

**`install.sh`** — One-shot installer (runs on router via SSH). Copies files, tests module loading, creates init script, installs addon page.

### Key paths on router
- `/opt/amneziawg/` — module, tool, config, client list, geo data
- `/jffs/addons/amneziawg/` — addon script + ASP page
- `/jffs/configs/dnsmasq.conf.add` — domain-based routing rules (tagged with `### AmneziaWG`)
- `/jffs/addons/custom_settings.txt` — Merlin settings store (all keys prefixed `awg_`)

### Routing model
Three policies per device: `vpn_all` (ip rule → table 200), `vpn_geo` (iptables fwmark 0x100 + ipset match → table 200), `direct`. Default policy applies to unlisted devices. Route table 200, priority 100, fwmark 0x100.

## Shell scripting notes

All router-side scripts must be POSIX sh (busybox ash) — no bashisms. The router runs BusyBox with limited coreutils.

---
> Source: [r0otx/asuswrt-merlin-amneziawg](https://github.com/r0otx/asuswrt-merlin-amneziawg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->

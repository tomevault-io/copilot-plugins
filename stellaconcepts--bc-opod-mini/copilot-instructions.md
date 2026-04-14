## bc-opod-mini

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OPODmini is an edge computing device for Blue Carbon environmental monitoring. This repository contains OS setup scripts and Docker stack configuration for deploying on Raspberry Pi CM4 (Seeed Studio reComputer R1000) running Raspberry Pi OS Trixie Lite (64-bit). Hardware interfaces include RS-485, RS-232, USB, GPIO, and 4G LTE (Quectel EC25).

## Commands

```bash
# Deploy the Docker stack
cd stack/ && docker compose up -d

# View service logs
docker compose logs -f nodered

# Check 4G modem status
lsusb | grep -i quectel
modem-power status  # Check modem power state
modem-power off     # Turn off modem (saves ~0.3W)
modem-power on      # Turn on modem

# Tailscale VPN
sudo tailscale status
sudo tailscale up --accept-dns=false  # Re-authenticate
```

## Architecture

### Setup Scripts (scripts/)

Single-stage provisioning system for Raspberry Pi OS Trixie Lite (64-bit):

- **`setup.sh`**: Main orchestrator. Collects Tailscale auth key and 4G APN interactively, then runs scripts 01-10 in sequence.

| Script | Purpose |
|--------|---------|
| `01_setup_gpiod.sh` | GPIO daemon for 4G modem power control |
| `02_setup_4g_modem.sh` | 4G modem ECM mode configuration |
| `03_install_tailscale.sh` | Tailscale VPN installation |
| `04_configure_dns.sh` | DNS configuration |
| `05_install_docker.sh` | Docker installation |
| `06_configure_docker_user.sh` | Docker user permissions |
| `07_setup_cpu_governor.sh` | CPU powersave governor for reduced power consumption |
| `08_optimize_power.sh` | Disable services, 100Mbps ethernet, USB autosuspend |
| `09_setup_modem_power.sh` | Modem power control utility (GPIO6) |
| `10_setup_modem_switch.sh` | GPIO17 input controls modem power |

Scripts are modular and can be run independently. They use shared logging to `/var/log/opodmini-setup.log` and environment variables (`NEW_USERNAME`, `TAILSCALE_AUTH_KEY`, `MODEM_APN`).

### Docker Stack (stack/)

| Service | Port | Purpose |
|---------|------|---------|
| Node-RED | 1880 | Flow-based automation, serial device communication |
| Fail2ban | - | Intrusion prevention (host network mode) |

Node-RED has access to serial devices (`/dev/ttyACM0`, `/dev/ttyACM1`) and MQTT certificates mounted at `./certs`.

### Hardware Interface

- **GPIO5/6**: 4G modem power control via gpiod
- **Serial**: RS-485 and RS-232 exposed as `/dev/ttyACM*`
- **RockBLOCK 9704**: Iridium satellite modem for areas without cellular coverage

### OS Configuration (OS/)

- **`config.txt`**: Custom boot configuration for CM4 with power-saving optimizations (disabled WiFi/BT, HDMI, reduced GPU memory, powersave governor)

## Conventions

- Script numbering (01_, 02_, etc.) indicates execution order
- All scripts require root/sudo and target Raspberry Pi OS Trixie Lite (64-bit)
- Docker services configured via `.env` file in `stack/` directory
- Default timezone: `Australia/Brisbane`

## References

### Online Documentation
- [RockBLOCK 9704 Python Library](https://pypi.org/project/rockblock9704/) - Official Ground Control library for Iridium satellite communication
- [RockBLOCK 9704 C API Reference](https://rock7.github.io/RockBLOCK-9704/rockblock__9704_8h.html) - Full function reference (rbBegin, rbSendMessage, rbReceiveMessage, etc.)

### Local Documentation (documents/)
PDFs requiring authentication are stored locally in the `documents/` folder. See `docs/REFERENCES.md` for the full list with descriptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stellaconcepts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stellaconcepts)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

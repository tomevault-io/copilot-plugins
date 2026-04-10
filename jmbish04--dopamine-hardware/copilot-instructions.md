## dopamine-hardware

> You are working in the `dopamine-hardware` repository. This is a Python-based edge hardware bridge running on a Raspberry Pi 4 (Debian/Raspberry Pi OS Lite 64-bit).

# 🤖 AI Agent Directives: Dopamine Hardware Bridge

## 🏗 Architecture Overview
You are working in the `dopamine-hardware` repository. This is a Python-based edge hardware bridge running on a Raspberry Pi 4 (Debian/Raspberry Pi OS Lite 64-bit). 
Its primary purpose is to act as a resilient, headless peripheral controller for the "Onion Tasker" Cloudflare Worker.

**Hardware Attached:**
- **Printer:** Epson TM-T20III Thermal Receipt Printer (USB: Vendor `04b8`, Product `0e28`).
- **Scanner:** Tera D5100 2D Barcode Scanner (Acts as a USB HID Keyboard).

**The 3-Tier Fallback Protocol:**
This bridge communicates with the Cloudflare Worker via three resilient methods:
1. **Tier 1 (VPC Push):** Flask server running on `0.0.0.0:8080`, exposed via Cloudflare Tunnel (`cloudflared`). The Worker makes direct HTTP requests.
2. **Tier 2 (WebSocket):** Background thread maintaining a persistent WSS connection to the Worker for pub/sub broadcasts.
3. **Tier 3 (REST Polling):** Background thread polling the Worker's pending jobs queue every 15 seconds to recover from network outages.

## 🛠 System Environment
- **Python Version:** 3.13
- **Virtual Environment:** Located at `/home/pi/dopamine-hardware/.venv`
- **Daemon:** Managed by systemd (`dopamine.service`).
- **Logging:** Dual-logging system. Logs are written to a local SQLite database (`dopamine_logs.db`) AND streamed asynchronously to the Cloudflare Worker telemetry endpoint.

## 🚨 Critical Agent Rules
1. **Never use `sudo pip` or `--break-system-packages`:** All Python dependencies MUST be installed inside the `.venv`.
2. **Thread Safety:** The printer is accessed by multiple threads (VPC, WS, REST). All printer interactions MUST be wrapped in the `printer_lock` to prevent USB bus collisions.
3. **No GUI Modules:** This is a headless environment. Never import `tkinter`, `matplotlib.pyplot` (with GUI backends), or anything requiring an X11 server.
4. **Resilience over Everything:** Hardware fails, cables get unplugged, networks drop. Every hardware or network call MUST have a strict `try/except` block that logs the failure gracefully without crashing the main threads.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmbish04)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jmbish04)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

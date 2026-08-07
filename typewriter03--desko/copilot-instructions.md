## desko

> Desko turns a spare Android phone (realme 3) into an always-on desk dashboard.

# Desko

Desko turns a spare Android phone (realme 3) into an always-on desk dashboard.
A lightweight Python server on a Windows PC pushes live data (now-playing music +
synced lyrics, VS Code git status, system performance, clock/weather) over WebSocket
to a single browser tab open on the phone via LAN.

**Before writing any code, read `IMPLEMENTATION_PLAN.md` in this directory.**
It is the single source of truth: architecture, data contracts, file layout,
milestone order, and acceptance criteria.

## Quick reference

- Backend: Python 3.11+, `aiohttp`, `winsdk`, `psutil`, `wmi` — one process, no DB.
- Frontend: dependency-free vanilla JS/CSS/HTML in `web/`, landscape-first (realme 3, 1520x720).
- Run: `python run.py` → server on `http://<pc-lan-ip>:7777`.
- Config: `config.json` (port, weather city, game process list, toggles).
- Demo mode: `python run.py --demo` fabricates all data so every scene is testable
  without real media/games/VS Code.

## Conventions

- Keep it light: <1% CPU idle, no build steps, no heavy frameworks.
- Graceful degradation: every collector must fail silently (hide its widget) when its
  data source is missing (no LibreHardwareMonitor, no VS Code extension, no internet).
- Windows-only collectors (GSMTC, WMI) must be import-guarded so the server still
  starts on other OSes for frontend development.

---
> Source: [typewriter03/Desko](https://github.com/typewriter03/Desko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->

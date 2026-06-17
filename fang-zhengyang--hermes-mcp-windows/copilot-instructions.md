## hermes-mcp-windows

> Windows Native MCP Server — operate Windows processes, services, registry, GUI, and system from Hermes Agent.


# Windows Native MCP Server

Exposes 26+ Windows system operations as MCP tools for Hermes Agent.

## Prerequisites

- Windows 10/11
- Python 3.12+
- Hermes Agent (any version with MCP support)

## Quick Start

### 1. Install dependencies (Windows side)

```powershell
pip install mcp pywin32 psutil pyautogui pillow
```

### 2. Start the server

```powershell
python path\to\windows_mcp_server.py --port 18765
```

### 3. Configure Hermes

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  windows:
    url: http://localhost:18765/mcp
    timeout: 60
```

### 4. Test the connection

```bash
hermes mcp test windows
```

### 5. Start a fresh Hermes session

All 26 `windows_*` tools will be available.

## Auto-start on Login

Place `setup/install_autostart.vbs` in the Windows Startup folder,
or double-click it — the server starts silently on every login.

Remove it with `setup/remove_autostart.vbs`.

## Available Tools

| Category | Tools |
|----------|-------|
| Process | `windows_process_list`, `windows_process_kill` |
| Service | `windows_service_list`, `windows_service_control` |
| Registry | `windows_registry_read`, `windows_registry_write` |
| System | `windows_system_info`, `windows_disk_info`, `windows_hotfix_list` |
| Environment | `windows_env_get`, `windows_env_set` |
| Notification | `windows_notification` |
| GUI | `windows_screenshot`, `windows_mouse_move/click`, `windows_type_text`, `windows_key_press` |
| Window | `windows_window_list`, `windows_window_focus` |
| Network | `windows_network_info`, `windows_network_stats` |
| File | `windows_file_info`, `windows_file_search` |
| Startup | `windows_startup_programs` |
| Execute | `windows_run_powershell`, `windows_run_cmd` |

---
> Source: [Fang-Zhengyang/hermes-mcp-windows](https://github.com/Fang-Zhengyang/hermes-mcp-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

## hypomux

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NetBooster is a Windows multi-network adapter concurrent download acceleration tool. It uses a SOCKS5 proxy server to intelligently distribute connections across multiple network adapters (Ethernet + Wi-Fi + Mobile Hotspot), achieving bandwidth aggregation for multi-threaded downloads.

**Core Technology Stack:**
- Python 3.10+ with PySide6 (Qt6) for GUI
- QFluentWidgets for Windows 11 Fluent Design UI
- asyncio for SOCKS5 proxy server
- psutil for real-time network traffic monitoring
- PowerShell for Windows network adapter configuration

## Development Commands

### Running the Application

```powershell
# Activate virtual environment
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run application (requires admin privileges - UAC prompt will appear)
python main.py
```

### Building Production Executable

```powershell
pip install nuitka zstandard PySide6-Fluent-Widgets
nuitka --standalone --onefile --enable-plugin=pyside6 --windows-console-mode=disable --windows-uac-admin --windows-icon-from-ico=assets/icon.ico --include-package-data=qfluentwidgets --include-data-dir=assets=assets --python-flag=-O --lto=yes main.py
```

## Architecture Overview

### Three-Layer Architecture

NetBooster implements a **dual-layer architecture** combining physical and application layer scheduling:

```
Application (Steam/IDM/qBittorrent)
         ↓ SOCKS5 Client → 127.0.0.1:1080
ProxyWorker (QThread + asyncio SOCKS5 Server)
         ↓ Round-robin connection distribution
Physical Layer Binding (socket.bind + IP_UNICAST_IF)
         ↓
Multiple Network Adapters → Internet (bandwidth aggregation)
```

### Key Modules

**Entry Point:**
- `main.py` - Application lifecycle: admin privilege check → QApplication creation → MainWindow initialization. Uses factory pattern to delay Qt imports until after QApplication exists.

**Core Proxy Engine:**
- `proxy_worker.py` - **ProxyWorker (QThread)**: Runs asyncio SOCKS5 server in separate thread to avoid blocking Qt event loop
  - **RoundRobinBalancer**: Distributes new connections across selected adapters in round-robin fashion
  - **Physical Layer Binding**: Uses `socket.bind((nic_ip, 0))` + `setsockopt(IPPROTO_IP, IP_UNICAST_IF, nic_index)` to force connections through specific adapters
  - **Async DNS Resolution**: Pre-resolves domains using `loop.getaddrinfo()` to avoid Windows single-adapter DNS deadlock
  - **Traffic Monitoring**: Real-time per-adapter download/upload speed and connection count using psutil

**UI Layer:**
- `ui/main_window.py` - **create_main_window()**: Factory function that returns MainWindow instance
  - **NetworkAdapterTableWidget**: Displays adapters with real-time connection counts
  - **ScanWorker (QThread)**: Background thread for adapter scanning
  - All Qt/qfluentwidgets imports are deferred until factory function is called (critical for avoiding "Must construct QApplication before QWidget" errors)

**Network Utilities:**
- `utils/network_utils.py` - PowerShell-based adapter management
  - `scan_network_adapters()` - Scans connected adapters with IPv4 addresses
  - `set_adapter_metric()` - Modifies adapter metric via PowerShell
  - **Critical**: All PowerShell commands use `InterfaceIndex` (numeric) instead of adapter names to avoid Chinese character encoding issues

**Application Configuration:**
- `utils/app_configurator.py` - Auto-configuration for Steam, IDM, qBittorrent proxy settings

### Signal Flow

**Startup:**
1. User clicks "一键加速" (Boost) → `_start_proxy()`
2. Creates ProxyWorker with selected adapters → `proxy_worker.start()`
3. ProxyWorker emits `started_ok` signal → UI updates status

**Runtime:**
1. SOCKS5 client connects → `_handle_client()` in ProxyWorker
2. RoundRobinBalancer assigns adapter → physical layer binding
3. Every second: `_traffic_monitor()` emits `traffic_signal` with per-adapter stats
4. UI receives signal → updates dashboard and connection counts

**Shutdown:**
1. User clicks "停止加速" (Stop) → `_stop_proxy()`
2. Calls `proxy_worker.stop()` → `loop.call_soon_threadsafe(event.set)`
3. Asyncio loop cancels all client tasks → emits `stopped` signal
4. UI resets to idle state

## Critical Implementation Details

### Physical Layer Binding (The Sacred Foundation)

From `proxy_worker.py:236-241`:
```python
upstream_sock.bind((nic['ip'], 0))
upstream_sock.setsockopt(socket.IPPROTO_IP, IP_UNICAST_IF, struct.pack("!I", nic['index']))
```

This dual binding is **non-negotiable**:
- `bind()` to adapter's IPv4 address
- `setsockopt()` with `IP_UNICAST_IF` (31) to force interface index

**Do not modify** this logic without testing on multiple adapters.

### Chinese Character Encoding

Windows PowerShell commands that use Chinese adapter names cause `subprocess` encoding crashes. Solution:

- **Always** use `InterfaceIndex` (numeric) in PowerShell commands
- Network adapter structure: `{'index': int, 'name': str, 'ip': str}`
- `name` is the Chinese alias (e.g., "以太网", "WLAN") - only for display
- `index` is the numeric InterfaceIndex - used for all PowerShell operations

### Qt Threading Model

**Main Thread:**
- Qt event loop (QApplication.exec())
- All UI updates (signals/slots)

**ScanWorker Thread:**
- Network adapter scanning via PowerShell
- Emits `scan_finished` signal back to main thread

**ProxyWorker Thread:**
- Dedicated asyncio event loop
- SOCKS5 server and all connection handling
- Uses `loop.call_soon_threadsafe()` for cross-thread communication
- Emits Qt signals that are thread-safe: `log_signal`, `traffic_signal`, `started_ok`, `stopped`, `error_signal`

**Never:**
- Call Qt UI methods from ProxyWorker thread directly
- Block the main thread with asyncio operations
- Use `asyncio.run()` in the main thread

### Adapter Selection Validation

From `ui/main_window.py:249-269`, `get_selected_adapters()` requires:
- Adapter must be checked in UI
- Adapter must have valid IPv4 address (handles comma-separated multi-IP cases)
- Returns `[{'index': int, 'name': str, 'ip': str}]` format expected by ProxyWorker

## Common Development Patterns

### Adding New Proxy Features

1. Add logic to ProxyWorker's `_handle_client()` method
2. Emit appropriate signal if UI feedback needed
3. Connect signal in MainWindow's `_start_proxy()` method
4. Handle signal with `@Slot` decorated method

### Modifying Network Adapter Operations

1. Add PowerShell command function in `utils/network_utils.py`
2. Use `_run_powershell_command()` helper with numeric InterfaceIndex
3. Return `Tuple[bool, str]` pattern: (success, message)
4. Handle in UI with InfoBar notifications

### Testing Proxy Changes

Use `proxy_core.py` as a standalone test script:
- Update `NIC_ETH` and `NIC_WLAN` with your adapter details
- Run `python proxy_core.py` directly
- Monitor console output for connection distribution
- This avoids full UI startup during proxy logic development

## Important Constraints

### Windows-Specific

- **Requires admin privileges**: UAC elevation happens in `main.py`
- **PowerShell encoding**: Use `encoding='mbcs', errors='replace'` in subprocess calls
- **Working directory**: `main.py` uses `os.path.dirname(os.path.abspath(__file__))` to prevent System32 redirection

### Multi-Threading

- ProxyWorker's asyncio loop runs in separate QThread
- Use `proxy_worker.stop()` for graceful shutdown (sets asyncio Event via `call_soon_threadsafe`)
- Always `wait()` for thread completion before setting `proxy_worker = None`

### UI State Management

During boost (acceleration):
- Disable adapter checkboxes (`set_checkboxes_enabled(False)`)
- Disable refresh button, port spinbox
- Change boost button to "停止加速" (Stop Acceleration)
- Enable all controls on stop

## Phase 3 Architecture Notes

Previous versions used "Metric Jiggling" (dynamically changing adapter metrics). **Phase 3 removed this approach** entirely in favor of:

- Pure SOCKS5 proxy-based distribution
- No modification of Windows routing tables during runtime
- Cleaner separation between UI and proxy core
- More stable under concurrent load

If you see references to `MetricJiggleWorker`, `set_dead_gateway_detection`, or `jiggle_adapter_metric` in old code, these are **deprecated** and should not be used.

## Recent Updates

### Phase 4: Auto System Proxy Configuration (2026-06-14)

**Problem Solved:** Users previously had to manually configure proxy settings for each application (Steam, IDM, qBittorrent, etc.), which was inconvenient and not user-friendly.

**Solution Implemented:** One-click automatic Windows system proxy configuration that makes NetBooster truly "plug-and-play".

#### Key Changes

**1. Enhanced `utils/app_configurator.py`:**
- Improved `configure_system_proxy()` method to use `socks=127.0.0.1:1080` format (natively supported by Windows 10/11)
- Added `get_current_system_proxy()` method to save original proxy state before configuration
- Added `auto_mode` parameter to distinguish between automatic silent configuration and manual configuration
- Uses `InternetSetOptionW` Windows API to notify system to refresh proxy settings (no browser restart needed)
- Proper handling of proxy state restoration: if proxy was disabled before start, it's disabled on stop; if it was enabled, original settings are preserved

**2. Updated `ui/main_window.py`:**

*UI Additions:*
- Added "自动配置系统代理" (Auto Configure System Proxy) checkbox next to port configuration (checked by default)
- Checkbox is disabled during acceleration to prevent mid-operation changes
- Added tooltip explaining supported applications

*Initialization:*
- Added `self.app_configurator` instance variable
- Added `self._original_proxy_enabled` and `self._original_proxy_server` to track original proxy state

*Startup Logic (`_start_proxy`):*
- Checks if auto system proxy checkbox is enabled
- Initializes `AppConfigurator` with current listen port
- Saves current system proxy state via `get_current_system_proxy()`
- Configures system proxy with `configure_system_proxy(auto_mode=True)`
- Logs configuration result to console
- Proxy server starts even if system proxy configuration fails (non-blocking)

*Shutdown Logic (`_stop_proxy`):*
- Automatically restores original system proxy settings
- If proxy was disabled before start, disables it
- If proxy was enabled before start, keeps original proxy server value
- Logs restoration result to console

*UI State Management:*
- `_enter_boosting_ui()`: Disables auto proxy checkbox during acceleration
- `_exit_boosting_ui()`: Re-enables auto proxy checkbox after stopping

*Fixed Missing Methods:*
- Added `on_config_steam_clicked()`: Steam one-click configuration
- Added `on_config_idm_clicked()`: IDM one-click configuration
- Added `on_config_qbt_clicked()`: qBittorrent one-click configuration
- Added `on_open_guide_clicked()`: Opens PROXY_GUIDE.md with default program

#### Supported Applications (Immediate Effect)

When "Auto Configure System Proxy" is enabled:
- **Browsers:** Chrome, Edge, Firefox (respects system proxy)
- **Download Managers:** IDM, Thunder (迅雷), Baidu NetDisk (百度网盘)
- **Gaming Platforms:** Steam (reads from IE proxy settings)
- **All applications** that respect Windows system proxy settings

#### User Experience

**Method 1: Auto System Proxy (Recommended, Easiest)**
1. Select network adapters
2. Ensure "自动配置系统代理" is checked ✓
3. Click "一键加速" (One-Click Boost)
4. All supported applications immediately benefit from multi-NIC acceleration!

**Method 2: Application-Specific Configuration (For Special Cases)**
- Click "Steam 一键配置" for Steam-specific shortcut
- Click "IDM 一键配置" for IDM registry modification
- Click "qBittorrent 一键配置" for qBittorrent config file modification

#### Technical Implementation Details

**Proxy Format:**
- Uses `socks=127.0.0.1:1080` format (not `http=` or `socks5://`)
- This is the Windows 10/11 native SOCKS5 system proxy format
- Set via registry: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings`
  - `ProxyEnable` = 1 (DWORD)
  - `ProxyServer` = "socks=127.0.0.1:1080" (REG_SZ)

**Windows API Notification:**
```python
import ctypes
internet_set_option = ctypes.windll.Wininet.InternetSetOptionW
internet_set_option(0, 39, 0, 0)  # INTERNET_OPTION_SETTINGS_CHANGED
internet_set_option(0, 37, 0, 0)  # INTERNET_OPTION_REFRESH
```

**State Preservation:**
- Original proxy state is captured in `_start_proxy()` before any modifications
- Restoration in `_stop_proxy()` checks if proxy was originally enabled
- This ensures user's pre-existing proxy configuration is never lost

#### Development Notes

**When adding new UI buttons with click handlers:**
1. Always define the corresponding `on_<action>_clicked()` method
2. Follow the pattern: create configurator → call method → show success/warning
3. Use `self.port_spinbox.value()` to get current listen port

**When modifying proxy configuration:**
- Always use `auto_mode=True` for automatic background operations
- Always use `auto_mode=False` for user-initiated manual configuration (shows detailed messages)
- Log all configuration/restoration actions to help users understand what's happening

**Testing checklist:**
1. Start NetBooster with auto proxy enabled → check Windows proxy settings
2. Open Chrome/Edge → verify traffic goes through SOCKS5 proxy
3. Stop NetBooster → verify proxy is disabled (or restored to original)
4. Test with pre-existing proxy configuration → verify it's preserved after stop

---
> Source: [Hypostasis-Cat/HypoMux](https://github.com/Hypostasis-Cat/HypoMux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

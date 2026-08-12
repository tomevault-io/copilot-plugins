## multiseat

> MultiSeat runs multiple simultaneous Moonlight game-streaming sessions on one Windows host. Each seat gets an isolated Windows account, virtual display (SudoVDA), virtual audio device, and a dedicated Apollo streaming instance, managed from a web dashboard.

# MultiSeat — Codebase Guide

MultiSeat runs multiple simultaneous Moonlight game-streaming sessions on one Windows host. Each seat gets an isolated Windows account, virtual display (SudoVDA), virtual audio device, and a dedicated Apollo streaming instance, managed from a web dashboard.

## Stack

- **Backend:** .NET 9 / ASP.NET Core Windows Service (`src/MultiSeat.Service`) — runs as SYSTEM
- **Frontend:** React + TypeScript dashboard (`src/MultiSeat.Dashboard`) — Vite build, served on port 9550
- **Shared:** `src/MultiSeat.Shared` — constants, port layout, shared types
- **Tests:** `src/MultiSeat.Tests`
- **InputHook DLL:** `src/MultiSeat.InputHook` — C++/CMake, keyboard/mouse session isolation
- **Solution:** `src/MultiSeat.slnx`

## Key Service Components

| Class | Responsibility |
|---|---|
| `SeatManager` | Seat lifecycle — provision / teardown |
| `SessionLauncher` | RDP loopback session creation, mstsc window management |
| `ApolloManager` | Per-seat Apollo process management |
| `VirtualDisplayManager` | SudoVDA virtual display attach/detach |
| `AudioRouter` | Virtual audio device assignment per seat |
| `InputRouter` | XInput/ViGEm controller routing |
| `HidHideConfigurator` | Controller cloaking via HidHide |
| `InputHookManager` | Keyboard/mouse session isolation (InputHook DLL) |
| `AccountManager` | Windows local account CRUD |
| `ApiServer` | ASP.NET Core HTTP API + WebSocket |

## Port Layout

Each seat reserves a block of 30 ports: `PortBase + (seat_index × 30)`. Default `PortBase = 48100`.
- Seat 0 → 48100–48129, Seat 1 → 48130–48159, etc.
- Apollo's per-seat offsets span `-5` (GFE HTTPS) to `+26` (RTSP) around the base.
- The base sits **above** a stock Apollo's block (~47979–48010, centered on the Moonlight default 47984) so MultiSeat coexists with a standalone Apollo — see "Coexistence with a standalone Apollo" below.

## Audio Device Layout

- Seat 0 → VB-CABLE basic "CABLE Input"
- Seat 1 → VoiceMeeter "VoiceMeeter Input"
- Seat 2 → VoiceMeeter "VoiceMeeter Aux Input"
- Seat 3 → VoiceMeeter "VoiceMeeter VAIO3 Input"

VoiceMeeter must be running — `AudioRouter` auto-starts it. Registered in `HKLM\Run` for auto-start at boot.

## Streaming behavior — resolution, audio, controller

Reflects fixes shipped 2026-07-24 (GitHub issues #11 / #10 / #9a):

- **Resolution follows the Moonlight client.** Each per-seat `sunshine.conf` sets Apollo's display-device keys — `dd_configuration_option = ensure_active`, `dd_resolution_option = auto`, `dd_refresh_rate_option = auto` (`ApolloConfigBuilder`). Apollo resizes the SudoVDA display to the mode the client requests on connect. The dashboard resolution is the SudoVDA creation/advertised default, **not** authoritative. **Requires the client's "Optimize game settings" (SOPS) enabled** — otherwise Apollo leaves the mode unchanged. Without the `dd_*` keys, `dd_configuration_option` defaults to `disabled` and Apollo never resizes the virtual display (it stays at the host/RDP surface size).
- **Seat audio does not hijack the host default.** The seat's virtual audio device is written as Apollo's `virtual_sink` (not `audio_sink`) with `keep_sink_default = disabled` + `auto_capture_sink = disabled`, and MultiSeat no longer runs `--set-default-render`. Windows has a **single machine-wide default output** shared by the console + all seats; Apollo still points the game at the sink during an active stream and restores the previous default afterward, without re-asserting it. Keep the client's "Play audio on host PC" **off** so `virtual_sink` is used. Not full isolation — true N-way seat audio needs per-app routing (`IAudioPolicyConfigFactory::SetPersistedDefaultAudioEndpoint`), not yet built.
- **Controller forwarding is native by default.** `EnableViGEmController` defaults **off** — Apollo forwards the Moonlight client's controller into the seat itself and MultiSeat creates no ViGEm pad. The dashboard shows the seat's Controller service as **"Native"** (not a down light) and the Input tab notes that XInput→seat assignment only applies when `EnableViGEmController` is on. `SeatServices.ControllerManaged` + `GET /api/input/mode` surface the mode.

## Install / Deploy

Two separate scripts — prereqs and service deploy are intentionally split:

```powershell
# Step 1: Install all prerequisites (drivers, audio devices, RDPWrap, etc.)
.\prerequisites\install-prerequisites.ps1
# Reboot if prompted, then re-run to confirm clean.
# Log: prerequisites\prereq.log

# Step 2: Build and deploy the MultiSeat service (run from scripts\)
.\scripts\install-service.ps1

# Remove the service
.\scripts\install-service.ps1 -Uninstall
```

## Key Runtime Paths

| Path | Purpose |
|---|---|
| `C:\Program Files\MultiSeat\` | Service install dir |
| `C:\Program Files\ApolloVibe\` | MultiSeat's own Apollo install (separate from any standalone `C:\Program Files\Apollo\`) |
| `C:\ProgramData\MultiSeat\` | Runtime data, Apollo configs, logs |
| `C:\ProgramData\MultiSeat\logs\` | `audio-helper.log` only — **not** the service log (see below) |
| `C:\ProgramData\MultiSeat\apollo\` | Per-seat Apollo config dirs (incl. per-seat `apollo.log`) |

## Where the service logs actually go

**The service writes no log files.** It's hosted via `AddWindowsService()`, whose default logging provider is the **Windows Event Log** — so service logs are in the Application log under two sources:

- `MultiSeat.Service` — application logging (`ILogger` categories; the detailed output)
- `MultiSeatService` — service lifecycle (started / stopped)

`scripts\show-logs.ps1` reads both, plus the helper and per-seat Apollo logs. Or directly:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='MultiSeat.Service'} -MaxEvents 50
```

`C:\ProgramData\MultiSeat\logs\` receives exactly one file, `audio-helper.log`, written by `AudioCaptureHelper` inside a seat session — so it stays **empty until a seat has run**. An empty folder there is normal and not a fault. (Two external reporters have gone looking for service logs there; the docs used to point them at it.)

## Coexistence with a standalone Apollo

MultiSeat is self-contained and **non-destructive**: it works out of the box whether or not the host already runs a standalone Apollo (e.g. for the main console account). Three guarantees keep the two from colliding:

1. **Own Apollo binary.** The prereq script installs ApolloVibe to `C:\Program Files\ApolloVibe\` (`Constants.DefaultApolloPath` / `MultiSeat:ApolloExePath`); an existing `C:\Program Files\Apollo\` is never touched.
2. **Own port range.** Default `PortBase = 48100`, above a stock Apollo's block — no runtime port conflict.
3. **Never kills a non-MultiSeat Apollo.** On startup `MultiSeatWorker.KillOrphanedApolloProcesses` reaps **only** Apollo processes MultiSeat launched, identified via WMI (`GetManagedApolloPids`) by executable path (under the ApolloVibe dir) or a MultiSeat per-seat config path on the command line. It no longer stops/disables `ApolloService`, and `install-service.ps1` leaves that service alone. (WMI failure → empty set → cleanup is skipped rather than risk killing an unrelated Apollo.)

## Shared game library & emulator netplay

Because each seat is its own Windows account, games/ROMs would otherwise be siloed per account and seats couldn't easily netplay. Two provisioning helpers address this (config in `MultiSeatOptions` / `appsettings.json`):

- **Shared game library** (`EnableSharedGameLibrary`, default on; `SharedGameLibraryDir`, default `C:\MultiSeatGames`). `SharedLibraryProvisioner.EnsureSharedLibraryAsync` runs once at startup: creates `…\SteamLibrary` + `…\ROMs` and grants `BUILTIN\Users` Modify via `icacls` (well-known SID `S-1-5-32-545`). Point each seat's Steam at the `SteamLibrary` folder (Settings → Storage) so a game an owning account already installed there isn't re-downloaded. ROMs go in `…\ROMs`.
- **Emulator netplay** (`EnableEmulatorNetplay`, default on). Each seat gets a deterministic, collision-free RetroArch host port from its own block: `seat.PortBase + Constants.OffsetRetroArchNetplay` (offset 13 → seat 0 = 48113, seat 1 = 48143…), surfaced as `SeatInfo.RetroArchNetplayPort` and opened in the firewall. Seats netplay each other over **loopback**: in one seat "Host", in another "Connect to Netplay Host" → `127.0.0.1:<host-seat-port>`. Netplay requires identical core **and** content — the shared ROM dir keeps file CRCs matching.
- **RetroArch auto-config** (`SeedRetroArchNetplayConfig`, default **off** — it writes a user file). When on, `RetroArchConfigSeeder` (an `IEmulatorConfigSeeder`) upserts `netplay_ip_port`, `netplay_public_announce=false`, `netplay_nat_traversal=false`, and `rgui_browser_directory` into each seat's `retroarch.cfg` during provisioning (mirrors the RustDesk seed in `SeatManager` step 2.5). Add Dolphin/PCSX2 by registering another `IEmulatorConfigSeeder` — no `SeatManager` change.

## Architecture Notes

- Service runs as **SYSTEM** — all process creation uses `CreateProcessAsUser` with appropriate session tokens.
- Sessions are created via **RDP loopback** (`127.0.0.2`) — the only reliable way to create new interactive sessions from Session 0.
- `mstsc.exe` is launched in the console session and kept alive (hidden via `SW_HIDE`) to hold the seat session in **Active** state for the lifetime of the seat. Do NOT disconnect — Apollo calls `QueryDisplayConfig` both at startup and when each Moonlight client connects; disconnected sessions return `ERROR_ACCESS_DENIED`.
- After Apollo creates its SudoVDA monitor, `SeatManager.ApplyDisplayIsolationAsync` runs in the seat session to set SudoVDA as the session primary and shrink the RDP virtual adapter to 640×480. This drops TermService CPU from ~70% to under 5% during streaming. The helper REQUIRES `seat.DisplayDevicePath` (Apollo's `output_name`); without it the isolation is skipped to avoid grabbing the wrong virtual display.
- Temp RDP files go to `C:\ProgramData\MultiSeat\` (not `%TEMP%`) so the console user's token can read them.
- `WTSQueryUserToken` returns a filtered (medium-integrity) token for admin accounts — `SessionLauncher` fetches the linked elevated token via `GetTokenLinkedToken` for SudoVDA IPC access.

## Launch-on-connect apps

Apollo creates the virtual Xbox 360 controller (ViGEm) only **while a Moonlight client is streaming**. Game launchers that scan controllers at startup (Steam Big Picture, EmulationStation, RetroBat) will not see a pad that appears afterwards — so if they're autostarted at login (e.g. a `Steam.lnk` in the seat user's Startup folder) they run before any stream and the controller is never detected.

Fix: don't autostart launchers in the seat. Instead let MultiSeat launch them **when a client connects**, after the pad exists. `OnConnectAppLauncher` (wired into `SessionHealthCheck`) tails each seat's `apollo.log` for `CLIENT CONNECTED` / `CLIENT DISCONNECTED` and launches/kills the configured apps on those edges.

Config lives in `appsettings.json` under `MultiSeat` — **empty by default (feature off); no apps are hardcoded**:

```jsonc
"LaunchOnConnectDelayMs": 4000,                 // wait after connect so the pad exists before launch
"KillLaunchOnConnectAppsOnDisconnect": false,   // true = kill the apps when the client disconnects
"LaunchOnConnect": [
  { "Path": "C:\\Program Files (x86)\\Steam\\steam.exe", "Arguments": "-bigpicture" }
  // { "Path": "...\\EmulationStation.exe", "WorkingDirectory": "..." }
]
```

Apps launch into the seat session via `ProcessInjector.LaunchInSessionAsync`. The list is global (applies to every seat). When the array is empty the watcher returns immediately — zero I/O, no overhead.

## Known Constraints

- NVIDIA consumer GPUs: 3–5 concurrent NVENC sessions max.
- RDPWrap breaks after Windows updates to `termsrv.dll` — re-run prereq script to refresh `rdpwrap.ini`.
- mstsc window for each seat must never be manually disconnected (session goes Disconnected, display APIs stop working).
- Single GPU only — multi-GPU not tested.
- Windows 11 build 26100+ / x64 only.
- VoiceMeeter audio drivers only register after a reboot post-install.
- Keyboard/mouse session isolation (`InputHookManager` + InputHook DLL) is **disabled by default and currently a no-op**. The low-level `WH_KEYBOARD_LL`/`WH_MOUSE_LL` hooks run in the SYSTEM service (Session 0), where `GetForegroundWindow()` returns NULL, so `ShouldPassThrough()` always passes — the filter never blocks. With the RDP-loopback design there is no cross-session K/M bleed anyway: physical input goes to the console session, and Moonlight input is `SendInput`'d inside the seat session. Re-enabling is only meaningful if the hook is re-architected to run inside the seat session.

## Required Prerequisites

- Apollo (Sunshine fork with multi-instance support) — NOT upstream Sunshine
- SudoVDA virtual display driver
- VB-CABLE basic (seat 0 audio)
- VoiceMeeter Potato (seats 1–3 audio)
- HidHide v1.5.230 (controller isolation)
- ViGEmBus v1.22.0 EXE — not MSI (virtual controller bus)
- RDPWrap (multi-session RDP on Windows Home/Pro)
- .NET 9 SDK
- Node.js 20+

---
> Source: [vibesoftwarecoder/MultiSeat](https://github.com/vibesoftwarecoder/MultiSeat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->

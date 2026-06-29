## squiggle

> Peer-to-peer LAN instant messenger. No central server — peers discover each other via multicast and communicate directly.

# Squiggle — Development Conventions

## Overview
Peer-to-peer LAN instant messenger. No central server — peers discover each other via multicast and communicate directly.

## Environment
- **.NET 9 SDK** required (no `global.json` — uses installed SDK)
- Primary platform: **Windows** (auto-start service uses Windows registry)
- Cross-platform via Avalonia, but some features are Windows-only

## Build

Squiggle targets **.NET 9** and uses the `dotnet` CLI. Build with:

```powershell
dotnet build Squiggle.sln
```

Run the UI project:

```powershell
dotnet run --project Squiggle.UI/Squiggle.UI.csproj
```

There are no automated tests in this repository.

## Architecture

Squiggle is a **peer-to-peer LAN instant messenger** built with WPF. There is no central server — peers discover each other via multicast and communicate directly.

### Layer diagram

```
Squiggle.UI          (WPF app — windows, viewmodels, plugin loader)
    ↓
Squiggle.Client      (Facade — ChatClient exposes buddy list, login, chat events)
    ↓
Squiggle.Core        (Networking — presence discovery, chat transport, message serialization)
    ↓
Squiggle.Infrastructure  (Shared abstractions — async pipes, serialization helpers)
Squiggle.Utilities       (Cross-cutting helpers used by all layers)
```

### Feature modules (plug into Client/UI)

| Project | Purpose | Key dependency |
|---|---|---|
| `Squiggle.FileTransfer` | P2P file transfer | Squiggle.Client |
| `Squiggle.VoiceChat` | Voice chat via NAudio + Speex | Squiggle.Client, NAudio |
| `Squiggle.Screenshot` | Screen capture & send | Squiggle.FileTransfer |
| `Squiggle.Translate` | Message translation | Squiggle.Client, Newtonsoft.Json |
| `Squiggle.History` | Chat history persistence | EntityFramework 6, System.Data.SQLite |
| `Squiggle.Multicast` | Multicast presence (standalone exe) | Squiggle.Core |
| `Squiggle.Bridge` | Cross-subnet/WAN bridging (standalone exe) | Squiggle.Core, protobuf-net |
| `Squiggle.Setup` | WiX MSI installer | WiX Toolset |

### Networking

- **Presence/discovery**: Multicast UDP (`UdpMulticastService`) or TCP fallback (`TcpMulticastService`) — peers announce themselves and listen for others on the LAN.
- **Chat transport**: Unicast message pipes between peers. `ChatHost` deserializes incoming messages and raises typed events; `ChatService` creates and manages chat sessions.
- **Serialization**: protobuf-net for wire protocol messages.
- **Bridge**: A standalone exe that forwards presence/chat messages between subnets for cross-LAN communication.

### UI patterns

- Avalonia UI 11 with Fluent theme and MVVM-style bindings. `MainWindow` uses a `ClientViewModel` and command pattern.
- Plugin/extension system: `Squiggle.Core` defines `IExtension`, `IMessageFilter`, `IMessageParser` interfaces; `PluginLoader` in the UI discovers and loads plugins at startup.
- Entry point is `Squiggle.UI/Program.cs` which enforces single-instance via a Mutex.
- Settings are persisted as JSON in `%AppData%/Squiggle/settings.json` via `SettingsService`.

### Migration status

- **Squiggle.UI**: Fully migrated to Avalonia 11
- **Squiggle.Translate**: Still uses WPF (`<UseWPF>true</UseWPF>`)
- All other projects are framework-agnostic (.NET class libraries)

## Key Files

| File | Purpose |
|------|---------|
| `Squiggle.UI/Program.cs` | Entry point — single-instance mutex + Avalonia startup |
| `Squiggle.UI/App.axaml.cs` | DI container setup, MainWindow creation, `/background` arg handling |
| `Squiggle.UI/MainWindow.axaml.cs` | Main UI — viewmodel init, tray icon, notifications |
| `Squiggle.Client/ChatClient.cs` | Facade — buddy list, login, chat events |
| `Squiggle.Core/Presence/UdpMulticastService.cs` | Multicast peer discovery |
| `Squiggle.Core/Chat/ChatHost.cs` | Incoming message deserialization + typed events |
| `Squiggle.UI/Services/WindowsAutoStartService.cs` | Windows-only auto-start via registry |

## Gotchas

- **Single-instance enforcement**: `Program.cs` uses a `Mutex` — launching a second instance will exit silently
- **WindowsAutoStartService registered unconditionally**: it's in DI regardless of OS, but decorated with `[SupportedOSPlatform("windows")]` — calling it on non-Windows will throw
- **Multicast requires LAN**: peers won't discover each other across subnets without the Bridge exe
- **No automated tests**: this repo has no test suite — validate changes by building and manual testing
- **Squiggle.Translate still uses WPF**: don't try to build it on non-Windows or without Windows Desktop SDK

## Conventions

- Events are initialized with empty delegates (`event ... = delegate { };`) to avoid null checks.
- Diagnostic logging uses Serilog (file sink in `logs/` folder).
- Translations live in `Squiggle.UI/Styles/Translations.axaml` as merged resource dictionaries.
- Settings persistence uses `System.Text.Json` to `%AppData%/Squiggle/settings.json`.

---
> Source: [hasankhan/Squiggle](https://github.com/hasankhan/Squiggle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->

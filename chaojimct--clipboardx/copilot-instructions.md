## clipboardx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

- **Development:** `dotnet run` (requires .NET 8 SDK, Windows)
- **Release publish (framework-dependent):** `dotnet publish ClipboardManager.csproj -c Release -r win-x64 -p:SelfContained=false -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p:IncludeAllContentForSelfExtract=true -o ./out/fdd`
- **Release publish (self-contained):** add `-p:SelfContained=true -p:EnableCompressionInSingleFile=true -o ./out/sc`
- **Native DLLs:** `powershell -ExecutionPolicy Bypass -File native/ShellNavigate/build.ps1` (requires MSVC Build Tools)
- **No test suite exists** in this repository.

## Product Flavors

Controlled by MSBuild property `ClipboardXProduct` (default `Full`):

| Flavor | Output | Defines |
|--------|--------|---------|
| `Full` | `ClipboardX.exe` | `CLIPX_FULL`, `CLIPX_CLIPBOARD`, `CLIPX_FILEJUMP` |
| `ClipboardOnly` | `ClipboardX-clipboard.exe` | `CLIPX_CLIPBOARD` |
| `FileJumpOnly` | `ClipboardX-filejump.exe` | `CLIPX_FILEJUMP` |

Conditional compilation (`#if CLIPX_CLIPBOARD`, `#if CLIPX_FILEJUMP`) gates features throughout the codebase. When modifying code, be aware that some files compile to multiple flavors.

## Architecture

**Windows system tray utility** -- not a standard WPF app. Uses `ShutdownMode="OnExplicitShutdown"`, a system tray `NotifyIcon`, and a non-activating popup (`WS_EX_NOACTIVATE`) that appears on hotkey press without stealing focus.

### Entry flow
`App.OnStartup()` → `AppPaths.Initialize()` → `AppSettings.Load()` → mutex check → theme → `PopupWindow` creation → global hotkeys → tray icon → background update check.

### Core subsystems
- **Clipboard history:** `ClipboardHistoryStore` (SQLite/WAL at `Data/clipboard_history.db`), `ClipboardEntry`, `ClipboardGate`
- **PopupWindow** (`Views/PopupWindow.xaml.cs`, ~264KB): Central UI with low-level keyboard/mouse hooks. Split into partial classes: `PopupWindow.KeyboardHook.cs`, `PopupWindow.MouseHook.cs`
- **File dialog jump:** `FileJump/FileDialogJumpHelper` + `FileManagerPathCollector` -- multi-strategy path reading from Explorer (COM `Shell.Application.Windows`), Total Commander (messages), XYplorer (WM_COPYDATA), Directory Opus, WPS, and generic `#32770` dialogs via UI Automation
- **Shell injection:** `FileJump/ShellDialogDeepNavigate` injects native DLL into `#32770` dialogs to call `IShellBrowser::BrowseObject` for deep navigation. The native C++ DLL is at `native/ShellNavigate/ClipboardXShellNavigate.cpp`
- **Everything IPC:** `Services/EverythingIpc` + `Services/ExplorerQuickFindController` -- uses voidtools `Everything64.dll` SDK for fast file search from Explorer
- **Settings:** `Models/AppSettings.cs` (~36KB) -- JSON persistence, `ShallowCopy()` for change tracking, `Load()` with migration

### Key files by size (largest = most complex)
1. `Views/PopupWindow.xaml.cs` (~264KB) -- main UI, hotkeys, clipboard monitoring, auto-sync
2. `Models/AppSettings.cs` (~36KB) -- all settings with defaults and serialization
3. `FileJump/FileDialogJumpHelper.cs` (~58KB) -- dialog detection, path reading, navigation strategies
4. `FileJump/FileManagerPathCollector.cs` (~46KB) -- Explorer/TC/XY/DOpus path collection
5. `Views/PopupWindow.KeyboardHook.cs` (~27KB) -- keyboard hook callback
6. `Interop/Win32Interop.cs` (~23KB) -- Win32 P/Invoke declarations
7. `FileJump/ShellDialogDeepNavigate.cs` (~22KB) -- DLL injection for dialog navigation

## Data Storage

- **Portable mode:** `Data\` folder next to exe (default)
- **Installed mode:** `%LocalAppData%\ClipboardX\`
- SQLite WAL for clipboard history, JSON for settings and custom file dialog rules

## ClipboardX/ Subdirectory

Separate **Avalonia cross-platform** solution (`ClipboardX.sln`), excluded from the main WPF build. Uses DI, CommunityToolkit.Mvvm, and platform abstractions. File dialog jump is Windows-only.

## Release Process

1. Update version in `ClipboardManager.csproj` `<Version>`
2. Update `CHANGELOG.md` with new version entry (GitHub Release auto-extracts from this)
3. Update `README.md` version history section
4. Commit, tag `v*`, push tag → triggers `.github/workflows/release.yml` CI
5. CI builds all 3 flavors, creates installers via Inno Setup, publishes GitHub Release

---
> Source: [chaojimct/clipboardx](https://github.com/chaojimct/clipboardx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->

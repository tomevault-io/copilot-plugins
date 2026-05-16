## ccusage-macos-menubar

> This is a minimal Tauri v2 application that runs as a menubar-only app on macOS. The app:

# CC Usage macOS Menubar App

## Overview

This is a minimal Tauri v2 application that runs as a menubar-only app on macOS. The app:
- Runs without a visible window on startup
- Shows only in the macOS menubar (system tray)
- Does not appear in the dock
- Displays real-time Claude Code usage data from ccusage CLI
- Shows daily usage costs per model (Opus 4, Sonnet 4, etc.)

## Architecture

### Backend (Rust)
- **src-tauri/src/lib.rs**: Main application logic
  - Sets up the system tray with dynamic menu items
  - Integrates with ccusage CLI via `npx ccusage@latest daily --json --breakdown`
  - Handles JSON parsing and data caching
  - Manages autostart functionality
  - Sets macOS activation policy to `Accessory` (no dock icon)

### Frontend (React)
- **src/App.tsx**: Minimal React component (not used in menubar-only mode)

### Configuration
- **Cargo.toml**: 
  - Enables `tray-icon`, `image-png` features
  - Includes `tokio` for async process execution
  - Includes `serde` for JSON parsing
- **tauri.conf.json**: 
  - Empty windows array (no window on startup)
  - Enables macOS private API for dock hiding

### Dependencies
- **ccusage CLI**: Required external dependency
  - Install with: `npm install -g ccusage` or use `npx ccusage@latest`
  - Provides Claude Code usage analytics

## Key Features

1. **System Tray Integration**
   - Icon appears in macOS menubar
   - Left-click shows menu with usage stats and options
   - No window interface - pure menubar app

2. **Current Session Display**
   - **Current session** shows the active 5-hour billing block
   - **Cost** and **Token counts** (Input/Output) displayed
   - **Session times** ("Started" and "Expires") shown as regular menu items
   - **Models used** header with each model listed separately
   - **Total cost** displayed in the menubar (e.g., $9.51) when active session exists
   - **"No active session"** displayed when no active block
   - **Refresh** (manually update all data)
   - **Launch on startup** (checkbox, toggles autostart)
   - **Quit** (with Cmd+Q shortcut)

3. **Error States**
   - **Install ccusage CLI** (clickable link to GitHub when ccusage not found)
   - **No usage data** (when no conversations today)
   - Graceful fallback to cached data on network issues

4. **Launch on Startup**
   - Uses Tauri's autostart plugin
   - Toggleable via menu checkbox
   - Works across macOS, Windows, Linux

5. **Data Integration**
   - **Current Session**: `npx ccusage@latest blocks --json --active`
   - Shows only the active 5-hour billing block
   - Caches data to handle network issues
   - Auto-formats model names (claude-opus-4-20250514 → "Opus 4")
   - Shows costs formatted as currency ($9.51)
   - Displays accurate session start and expiration times
   - Handles no active session gracefully

6. **Smart Refresh & Performance**
   - **Concurrent data fetching** - all time periods updated simultaneously using `tokio::join!`
   - **Smart caching** - only fetches new data if cache is older than 5 minutes
   - **No menu interruption** - menu stays open during refresh
   - **Manual refresh** forces immediate update of all time periods
   - **Fast startup** with cached data

7. **macOS Specific**
   - Uses `ActivationPolicy::Accessory` to hide from dock
   - Icon adapts to light/dark mode with `icon_as_template(true)`

## Build & Run

### Prerequisites
1. **Install ccusage CLI** (required for usage data):
   ```bash
   npm install -g ccusage
   # OR use npx (no installation needed):
   npx ccusage@latest --help
   ```

### Development
```bash
yarn
yarn tauri dev
```

### Production Build
```bash
yarn tauri build
```

## Testing

To verify the menubar behavior:
1. **First time setup**: Ensure ccusage CLI works: `npx ccusage@latest daily --json`
2. **Run the app**: `yarn tauri dev` - no window should appear
3. **Find the icon**: Look for the app icon in the macOS menubar
4. **Test menu**: Left-click the icon to see usage data
5. **Test refresh**: Click "Refresh" to update data
6. **Test error state**: If ccusage isn't available, click "Install ccusage CLI"
7. **Verify dock**: The app shouldn't appear in the dock

### Menu Examples

**Normal state** (with active session):
```
CCUsage
────────────────
Current session
Cost: $17.59
Tokens: In 6.5K / Out 5.5K
Started: 10:00 PM
Expires: 3:00 AM
────────────────
Models used
Opus 4
────────────────
☑ Launch on startup
Refresh
────────────────
Quit
```

**No active session state**:
```
CCUsage
────────────────
Current session
No active session
────────────────
☑ Launch on startup
Refresh
────────────────
Quit
```

**Benefits of Current Approach**:
- **Accurate session times** - shows actual 5-hour billing block start/end
- **Clear cost display** - total cost visible in menubar when session is active
- **No confusion** - displays only the active session, not multiple blocks
- **Graceful handling** - shows "No active session" when appropriate
- **Smooth user experience** - no lag or jitter

## Notes

- The app uses Tauri v2's tray icon API
- Window size is set to 400x300 when created
- The tray icon uses the default app icon from `src-tauri/icons/`
- Frontend can be expanded with any React components as needed

## GitHub Actions Release Workflow

The project includes a GitHub Actions workflow for automated releases.

### Using the Release Workflow
No Apple Developer account required. To create a release:

1. Go to the Actions tab on GitHub
2. Select "Release" workflow
3. Click "Run workflow"
4. Enter version (e.g., `1.0.0` - without v prefix)
5. The workflow will:
   - Update version in Cargo.toml and tauri.conf.json
   - Build for Apple Silicon (ARM64) only
   - Create a draft release with the DMG file
   - Note: Users will need to bypass Gatekeeper on first run

### Requirements
- Targets Apple Silicon Macs only (M1/M2/M3)
- No code signing (see "First Run Instructions" below)
- Requires Node.js installed on the user's machine for ccusage functionality

### First Run Instructions
Since the app is not code-signed, macOS may show "app is damaged and can't be opened" when downloading from GitHub releases.

**To fix this:**
```bash
# After moving the app to /Applications, run:
xattr -cr /Applications/ccusage-macos-menubar.app
```

**Alternative method:**
1. Right-click the app and select "Open"
2. Click "Open" in the security dialog

This only needs to be done once. The app is safe - macOS shows this for all unsigned apps downloaded from the internet.

### Manual Release Process
If you prefer to build and release manually:
```bash
# Build for Apple Silicon (ARM64)
yarn tauri build -- --target aarch64-apple-darwin

# Output will be in:
# src-tauri/target/release/bundle/dmg/
```

---
> Source: [6/ccusage-macos-menubar](https://github.com/6/ccusage-macos-menubar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->

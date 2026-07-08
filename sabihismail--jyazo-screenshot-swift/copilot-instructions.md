## jyazo-screenshot-swift

> **IMPORTANT: After code changes, Claude should automatically:**

# ScreenShot macOS App - Development Workflow

## Quick Development Cycle

**IMPORTANT: After code changes, Claude should automatically:**
1. Build the app
2. Kill any running instances
3. Open the fresh build

**Do not ask the user to build or open - do it automatically.**

### Automatic Build & Run Command
```bash
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -scheme Jyazo -configuration Debug build 2>&1 | grep "BUILD" && \
killall ScreenShot 2>/dev/null; sleep 1; open ~/Library/Developer/Xcode/DerivedData/ScreenShot-*/Build/Products/Debug/Jyazo.app
```

### Manual Testing (if needed)

#### 1. Build the App
```bash
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -scheme Jyazo -configuration Debug build
```

#### 2. Reset Permissions (TCC Database)
```bash
tccutil reset ScreenCapture arkaprime.Jyazo
tccutil reset Accessibility arkaprime.Jyazo
```

#### 3. Open App with Debug Logs

**Option A: Via Xcode (recommended for debugging)**
1. Open Xcode: `open -a Xcode <repo-root>/ScreenShot.xcodeproj`
2. Product → Run (⌘R)
3. Console window shows live debug output
4. When app restarts after permission changes, logs continue in Xcode console

**Option B: Via Terminal with Console Streaming**
```bash
# Terminal 1: Watch logs in real-time
log stream --predicate 'process == "ScreenShot"' --level debug

# Terminal 2: Open the app
open ~/Library/Developer/Xcode/DerivedData/ScreenShot-*/Build/Products/Debug/Jyazo.app
```

**Option C: Via macOS Console.app**
1. Open Console.app: `open /Applications/Utilities/Console.app`
2. Search for: "ScreenShot"
3. Open the app in another window
4. Logs appear in real-time

### 4. Testing Permission Flow

1. **App opens** → See debug logs in Xcode or Console
2. **Click "Request Permissions"** → Permission dialog appears
3. **Grant/Deny permission** → Logs show what happened
4. **Click "Reset & Restart"** → App resets TCC entry and restarts
5. **Logs continue** (if using Xcode or `log stream`) → Permission dialog appears again

### Quick One-Liner Script

Build + reset + open in Xcode with logs:
```bash
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -scheme Jyazo -configuration Debug build && \
tccutil reset ScreenCapture arkaprime.Jyazo && \
tccutil reset Accessibility arkaprime.Jyazo && \
open -a Xcode <repo-root>/ScreenShot.xcodeproj
```

Then press ⌘R in Xcode to run and see live logs.

## Bundle Information

- **Bundle ID**: `arkaprime.Jyazo`
- **Build Output**: `~/Library/Developer/Xcode/DerivedData/ScreenShot-*/Build/Products/Debug/Jyazo.app`

## Resetting Permissions

### From Code (In-App Menu)
- Button: **"Reset & Restart"** - Resets all permissions and restarts the app
- Button: **"Request Permissions"** - Triggers permission request dialog

### From Terminal

**Quick way (recommended for full reset):**
```bash
bash <repo-root>/clear_permissions.sh
```
This clears ALL TCC permissions (Accessibility, Screen Recording). You'll be prompted for your sudo password.

**Manual reset (if needed):**
```bash
# Reset screen recording only
tccutil reset ScreenCapture arkaprime.Jyazo

# Reset accessibility only
tccutil reset Accessibility arkaprime.Jyazo

# Reset all
tccutil reset ScreenCapture arkaprime.Jyazo && \
tccutil reset Accessibility arkaprime.Jyazo
```

## Logging System

All logs go to **two places**:
1. **Xcode Console** - Live output while running (if using `⌘R`)
2. **Log File** - Persistent record in `~/Library/Application Support/ScreenShot/Logs/`

Each app launch creates a new timestamped log file.

### View Logs
- **In Menu**: "Open Log File" or "Open Logs Directory"
- **Terminal**: `ls ~/Library/Application\ Support/ScreenShot/Logs/`
- **Tail latest**: `tail -f ~/Library/Application\ Support/ScreenShot/Logs/ScreenShot_*.log`

### Debug Log Tags

Watch for these in the logs:
- `[CAPTURE]` - Screenshot capture events
- `[UPLOAD]` - Upload flow
- `[OAUTH]` - OAuth authentication
- `[AUTH]` - Local auth server
- `[PERMISSIONS]` - Permission requests
- `[GIF]` - GIF recording

## OAuth Testing

1. Ensure server is running: `arkapri.me` (check `/api/authenticate` endpoint works)
2. Update server URL in Preferences if needed (default: `https://arkapri.me`)
3. Click "Capture Region" to trigger OAuth flow
4. Browser opens in Chrome (auto-opens with localhost:52805 callback)
5. Check logs for token receipt: `[AUTH] Token received, expires: ...`

## Common Issues

### "Socket failed to start listening within 10.0s"
- Port 52805 is already in use
- Solution: Kill existing Jyazo processes: `killall Jyazo`
- Or reset: `tccutil reset ScreenCapture arkaprime.Jyazo && tccutil reset Accessibility arkaprime.Jyazo`

### Permission Dialog Not Appearing After Reset
- Use "Reset & Restart" button in app menu
- Or manually reset via terminal + restart app
- Xcode's Run (⌘R) will pick up new build automatically

### Chrome Not Opening
- Ensure Chrome is installed (`/Applications/Google Chrome.app`)
- App falls back to default browser if Chrome unavailable
- Check logs: `[OAUTH] Opening Chrome for OAuth`

---
> Source: [sabihismail/Jyazo-Screenshot-Swift](https://github.com/sabihismail/Jyazo-Screenshot-Swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->

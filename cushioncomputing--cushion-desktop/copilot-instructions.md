## cushion-desktop

> A native desktop wrapper for the Cushion web application built with Tauri v2. This provides a native experience for macOS and iOS by embedding the web application in a native webview with deep OS integration.

# Cushion Desktop App - Developer Guide

## Project Overview

A native desktop wrapper for the Cushion web application built with Tauri v2. This provides a native experience for macOS and iOS by embedding the web application in a native webview with deep OS integration.

**Version:** 0.1.0
**Framework:** Tauri v2
**Language:** Rust + JavaScript

## Key Features

- ✅ Native macOS and iOS app shell
- ✅ Magic link authentication (desktop-aware)
- ✅ Native push notifications
- ✅ Deep link handling (`cushion://` URL scheme)
- ✅ System theme support (light/dark mode)
- ✅ Zoom controls (Cmd +/- /0)
- ✅ External link handling
- ✅ Window state persistence
- ✅ Hot reload during development

## Project Structure

```
cushion-desktop/
├── src-tauri/                 # Rust backend
│   ├── src/
│   │   ├── main.rs           # Application entry point
│   │   ├── lib.rs            # Core application logic and commands
│   │   └── commands/         # (TODO) Tauri command modules
│   ├── icons/                # App icons
│   │   ├── icon.icns         # Production icon
│   │   └── dev-icon.icns     # Development icon
│   ├── capabilities/         # Tauri permissions
│   ├── Cargo.toml           # Rust dependencies
│   └── tauri.conf.json      # Tauri configuration
├── build-config.js           # Dynamic build configuration
├── create-dev-dmg.sh         # DMG creation script
├── generate-icons.sh         # Icon generation script
└── package.json             # NPM scripts and dependencies
```

## NPM Scripts

### Development
- `npm run dev` or `npm run dev:desktop` - Start desktop app with hot reload
- `npm run dev:ios` - Start iOS simulator
- `npm run test:deep-links` - Test deep link handling

### Building
- `npm run build` or `npm run build:desktop` - Production build (release mode)
- `npm run build:dev` - Quick dev build (debug mode, creates .dmg)
- `npm run build:test` - Test build (points to localhost:3000)
- `npm run build:ios` - iOS production build

### Utilities
- `./generate-icons.sh` - Generate .icns files from 1024x1024 source image

## Architecture

### Webview Integration

**Development Mode:**
- Connects to web-app dev server at `http://localhost:3000`
- Hot reload enabled
- Must start web-app separately: `cd ../web-app && npm run dev`

**Production Mode:**
- Configuration determined by `build-config.js` at build time
- Reads from `../web-app/package.json` for dependencies
- Creates optimized bundles

### Build Modes

The app supports three build configurations via environment variables:

1. **Production** (default)
   - Bundle ID: `com.cushion.desktop`
   - Deep links: `cushion://`
   - Optimized release build
   - Command: `npm run build`

2. **Development**
   - Bundle ID: `com.cushion.desktop.dev`
   - Deep links: `cushion-dev://`
   - Debug build, .app bundle only, wrapped in .dmg
   - Command: `TAURI_BUILD_DEV=true npm run build:dev`

3. **Test**
   - Points to `localhost:3000` for debugging
   - Debug build with dev server URL
   - Command: `TAURI_BUILD_TEST=true npm run build:test`

## Rust Backend (src-tauri/)

### Main Components

#### lib.rs
Contains all Tauri commands and application setup logic:

**Tauri Commands:**
- `greet(name: &str)` - Test command
- `show_notification(title, body)` - Show system notification
- `check_notification_permission()` - Check notification permissions
- `get_user_agent()` - Get custom user agent (detects dev vs prod)
- `show_main_window()` - Show and focus the main window
- `is_window_visible()` - Check if window is visible
- `is_window_focused()` - Check if window is focused
- `is_window_minimized()` - Check if window is minimized
- `set_zoom_level(zoom: f64)` - Set webview zoom level
- `open_url(url: String)` - Open URL in default browser

**macOS-specific Functions:**
- `is_dark_mode()` - Detect system dark mode
- `update_window_background_for_theme()` - Update window colors for theme

### Window Configuration

**Title Bar:**
- Style: `TitleBarStyle::Overlay`
- Traffic light position: `(15.0, 20.0)`
- Hidden title bar with transparent background

**Default Size:**
- Initial: 1200x800
- Minimum: 800x600
- Resizable: Yes

**Window Behavior:**
- Close button hides window instead of destroying it
- Dock icon click shows hidden window
- Window state persists between sessions (via tauri-plugin-window-state)
- Theme changes update background color dynamically

### Initialization Scripts

**Swipe Prevention:**
```javascript
document.body.style.overscrollBehaviorX = 'none';
document.body.style.overscrollBehaviorY = 'none';
```

**Zoom Controls:**
- Cmd/Ctrl + `=` - Zoom in
- Cmd/Ctrl + `-` - Zoom out
- Cmd/Ctrl + `0` - Reset zoom

**External Link Handling:**
- Links to different origins open in default browser
- Same-origin links navigate within webview

## Dependencies

### Rust (Cargo.toml)
- `tauri` v2 - Main framework
- `serde` - JSON serialization
- `cocoa` - macOS native APIs
- `objc` - Objective-C runtime
- `tauri-plugin-notification` - Notifications
- `tauri-plugin-opener` - Open URLs
- `tauri-plugin-deep-link` - Deep link handling
- `tauri-plugin-window-state` - Window state persistence

### JavaScript (package.json)
- `@tauri-apps/api` ^2.8.0 - Tauri frontend API
- `@tauri-apps/plugin-notification` ^2.3.1 - Notification plugin

## Authentication Flow

### Magic Link Support

1. User clicks "Sign in with email" in desktop app
2. Backend detects Tauri user agent and sends desktop-specific magic link
3. Magic link uses `cushion://auth/success` scheme
4. Link opens desktop app via deep linking
5. App handles deep link event and completes authentication

**User Agent Detection:**
```rust
// lib.rs get_user_agent() command
// Returns: "Mozilla/5.0 ... CushionDesktop/dev" or "CushionDesktop/prod"
```

**Deep Link Handling:**
```rust
// Listens for "deep-link://new-url" events
// Shows/focuses window when deep link received
// Emits "deep-link" event to frontend with URL
```

## Theme Support

### Light Mode
- Background: `rgb(250, 250, 250)` / `hsl(0, 0%, 98%)`
- Border: `rgba(18, 18, 18, 0.1)`

### Dark Mode
- Background: `rgb(15, 15, 15)` / `hsl(0, 0%, 6%)`
- Border: `rgba(255, 255, 255, 0.1)`

Theme changes are detected automatically via `WindowEvent::ThemeChanged` and update the window background color immediately.

## Development Workflow

### 1. Start Development Environment

**Terminal 1 (Web App):**
```bash
cd ../web-app
npm run dev  # Starts on localhost:3000
```

**Terminal 2 (Desktop App):**
```bash
cd cushion-desktop
npm run dev  # Opens Tauri app connected to localhost:3000
```

### 2. Testing Builds

**Quick Dev Build:**
```bash
npm run build:dev
# Creates: src-tauri/target/debug/bundle/macos/*.dmg
# Fast debug build for testing
```

**Production Build:**
```bash
npm run build
# Creates: src-tauri/target/release/bundle/macos/*.dmg
# Full optimized release build
```

### 3. Testing Deep Links

```bash
npm run test:deep-links
# Opens: cushion://auth/success?token=test&callbackUrl=%2Fdashboard
```

### 4. Icon Generation

```bash
./generate-icons.sh
# Interactive script to generate .icns from 1024x1024 source
# Prompts for source path and dev/prod selection
```

## Common Tasks

### Adding a New Tauri Command

1. Define command in the appropriate `src-tauri/src/commands/*.rs` module:
```rust
#[tauri::command]
pub fn my_command(param: String) -> Result<String, String> {
    // Implementation
    Ok("result".to_string())
}
```

2. Export from `commands/mod.rs` if in a new file

3. Add to invoke_handler in `lib.rs`:
```rust
.invoke_handler(tauri::generate_handler![
    // existing commands...
    commands::mymodule::my_command
])
```

4. Call from frontend:
```javascript
import { invoke } from '@tauri-apps/api/core';
const result = await invoke('my_command', { param: 'value' });
```

## Adding New Features

When adding new functionality, place code in the appropriate module:

| Feature Type | Location |
|-------------|----------|
| Tauri command | `commands/<category>.rs` |
| Window creation/behavior | `window/mod.rs` |
| Window events | `window/events.rs` |
| Update system | `updater/` |
| Menu items (macOS) | `menu/macos.rs` |
| Notifications | `notifications/` |
| Theme/appearance | `theme.rs` |
| macOS-specific | Use `#[cfg(target_os = "macos")]` |

**DO NOT** add to `lib.rs` - it's only for entry point, plugin registration, and setup orchestration.

### Debugging

**macOS Console Logs:**
- Rust println!() output appears in terminal
- JavaScript console.log() - Right-click → Inspect Element

**iOS Debugging:**
- Safari → Develop → Simulator → [Your App]

### Clearing Build Cache

```bash
cd src-tauri
cargo clean
cd ..
npm run build:dev
```

## Build Outputs

### macOS
- `.app` bundle: `src-tauri/target/{debug|release}/bundle/macos/`
- `.dmg` installer: `src-tauri/target/{debug|release}/bundle/dmg/`

### iOS
- Xcode project: `src-tauri/gen/apple/`
- `.ipa` distribution: Via Xcode → Archive

## Troubleshooting

### Build Issues

**Problem:** Build config script fails
- Check `../web-app/package.json` exists and is valid
- Verify Node.js can run `build-config.js`

**Problem:** Rust compilation errors
- Update Rust: `rustup update`
- Clean build: `cd src-tauri && cargo clean`
- Check Xcode Command Line Tools: `xcode-select --install`

**Problem:** Icon not showing
- Verify `.icns` files exist in `src-tauri/icons/`
- Rebuild after adding new icons
- For dev builds, use `dev-icon.icns`

### Runtime Issues

**Problem:** Window doesn't show on launch
- Check console for errors
- Verify webview URL is accessible
- For dev builds, ensure web-app is running on localhost:3000

**Problem:** Deep links not working
- Test URL scheme registration: `npm run test:deep-links`
- Check `tauri.conf.json` has correct bundle identifier and deep link config

**Problem:** Notifications not working
- macOS: Check System Settings → Notifications → Cushion
- First notification shows permission prompt

## Best Practices

### Code Organization
- Keep Tauri commands focused and single-purpose
- Use Result<T, String> for error handling
- Document public functions with doc comments
- Separate concerns into modules (TODO: refactor lib.rs)

### Window Management
- Use `hide()` instead of `close()` to preserve state
- Listen for `WindowEvent::CloseRequested` to prevent exit
- Handle `RunEvent::Reopen` for dock icon clicks

### Platform-Specific Code
```rust
#[cfg(target_os = "macos")]
{
    // macOS-only code
}

#[cfg(target_os = "ios")]
{
    // iOS-only code
}
```

### Error Handling
```rust
// Convert errors to strings for Tauri
.map_err(|e| e.to_string())?
```

## Future Improvements

### Code Organization (TODO)
- [ ] Split `lib.rs` into modules:
  - `commands/` - Tauri command functions
  - `window/` - Window management
  - `theme.rs` - Theme detection and styling
  - `auth.rs` - Authentication helpers
- [ ] Add unit tests for Rust functions
- [ ] Add integration tests

### Features (TODO)
- [ ] Auto-updater integration
- [ ] Tray icon with menu
- [ ] Keyboard shortcuts configuration
- [ ] Multi-window support
- [ ] App Store distribution
- [ ] Windows support

## References

- [Tauri Documentation](https://tauri.app/v2/guides/)
- [Tauri Commands](https://tauri.app/v2/develop/calling-rust/)
- [Tauri Window Management](https://tauri.app/v2/reference/javascript/api/namespaces/window/)
- [Tauri Deep Links](https://tauri.app/v2/guides/deep-linking/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cushioncomputing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cushioncomputing)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

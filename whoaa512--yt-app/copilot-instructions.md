## yt-app

> Native macOS YouTube wrapper — Swift, AppKit, WKWebView. No storyboards.

# YTApp

Native macOS YouTube wrapper — Swift, AppKit, WKWebView. No storyboards.

## Build

```bash
./build.sh
```

Or manually: `cd YTApp && xcodebuild -scheme YTApp -configuration Debug SYMROOT=/Users/cjw/code/yt-app/build CODE_SIGN_IDENTITY="-" CODE_SIGN_STYLE=Manual CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO build`

**Important**: Always use `SYMROOT=/Users/cjw/code/yt-app/build` — the app is launched from `build/Debug/YTApp.app`, not DerivedData.

**Passkeys/WebAuthn**: Requires `com.apple.developer.web-browser` entitlement which needs a real Apple Development certificate (ad-hoc signing can't use restricted entitlements). When a cert is available, `build.sh` should re-sign with `YTApp/YTApp/YTApp.entitlements`.

## File Map

### Swift
| File | Role |
|------|------|
| `MainWindowController.swift` | Window, layout, all WKWebView delegates, message handler dispatch |
| `TabManager.swift` | Tab lifecycle, shared WKWebViewConfiguration, JS injection, suspension |
| `Tab.swift` | Tab model (URL, title, webView, suspension state) |
| `ToolbarView.swift` | Playback/nav toolbar with hover buttons, speed controls |
| `AddressBarView.swift` | URL bar with back/forward |
| `QueueManager.swift` | Queue model (singleton, UserDefaults persistence) + ThumbnailCache |
| `QueueSidebarView.swift` | NSTableView sidebar with drag-to-reorder, thumbnails |
| `HistoryManager.swift` | SQLite history |
| `HistoryViewController.swift` | History panel UI |
| `MediaKeyHandler.swift` | MPNowPlayingInfoCenter + remote commands |
| `JSConsoleWindowController.swift` | Dev console for JS evaluation |
| `Settings.swift` | UserDefaults wrapper (playback rate, theater mode, etc.) |
| `URLRouter.swift` | Domain allowlist, external URL handling |
| `YTWebView.swift` | WKWebView subclass with right-click context menu (Summarize) |
| `SummarySidebarView.swift` | Right-side drawer showing video summaries |
| `DownloadManager.swift` | yt-dlp download backend, SQLite metadata, progress tracking |
| `OfflineLibraryView.swift` | Full-page inline grid of downloaded videos (YouTube-style cards) |
| `OfflinePlayerView.swift` | Native AVPlayerView player with playlist nav, PiP, speed controls |

### JavaScript (`YTApp/YTApp/JS/`)
| File | Injection | Role |
|------|-----------|------|
| `MediaBridge.js` | documentEnd | Polls video state → `mediaBridge` handler |
| `URLObserver.js` | documentEnd | SPA navigation tracking → `urlChanged` handler |
| `DurationExtractor.js` | evaluated on demand | Extracts video duration for history |
| `QueueInterceptor.js` | documentStart | Hijacks "Add to queue" → `queueBridge` handler |
| `TheaterMode.js` | documentStart | Persists `wide=1` cookie for theater mode |
| `SummarizeContext.js` | documentEnd | Tracks right-clicked renderer for video URL extraction |

### Message Handlers (Swift ↔ JS bridge)
| Handler | Direction | Purpose |
|---------|-----------|---------|
| `mediaBridge` | JS→Swift | Video playback state (paused, ended, time, title) |
| `urlChanged` | JS→Swift | SPA URL changes for address bar + history |
| `queueBridge` | JS→Swift | Intercepted queue additions with video metadata |
| `consoleLog` | JS→Swift | Debug logging to JS console window |
| `theaterChanged` | JS→Swift | Theater mode toggle state sync |

## Adding Files

### New JS injection
1. Create `.js` in `YTApp/YTApp/JS/`
2. pbxproj: add `BB______` (PBXFileReference) + `AA______` (PBXBuildFile)
3. pbxproj: add to JS group (`EE000003`) + Resources build phase
4. `TabManager.swift`: inject via `WKUserScript` in `sharedConfiguration`
5. If it posts messages: register handler in `MainWindowController.windowDidLoad`, handle in `userContentController(_:didReceive:)`

### New Swift file
1. Create `.swift` in `YTApp/YTApp/`
2. pbxproj: add `BB______` (PBXFileReference) + `AA______` (PBXBuildFile)
3. pbxproj: add to YTApp group (`EE000002`) + Sources build phase

### Pbxproj IDs
Sequential, prefixed: **AA** (build files), **BB** (file refs), **CC** (products), **DD** (frameworks phase), **EE** (groups), **FF** (build phases/project), **GG** (build configs). Check highest existing number and increment.

## YouTube Gotchas

**SPA navigation**: Clicking videos doesn't trigger WKNavigationDelegate. `URLObserver.js` hooks `pushState`, `replaceState`, `popstate`, and `yt-navigate-finish`.

**DOM scraping is brittle**: YouTube's markup changes. Key patterns:
- Video metadata lives in `ytd-*-renderer` elements (`ytd-rich-item-renderer`, `ytd-compact-video-renderer`, `ytd-video-renderer`, `ytd-rich-grid-media`)
- Title: `#video-title` or `aria-label` on title link
- Channel: `ytd-channel-name #text` or `#channel-name a`
- Duration: `ytd-thumbnail-overlay-time-status-renderer span`
- Views/date: `#metadata-line span` or `.inline-metadata-item`
- **Context menus lose DOM context** — the `yt-action` event fires after the popup closes. Track the renderer via `contextmenu` listener in capture phase.
- **Always have a fallback**: oEmbed API (`/oembed?url=...&format=json`) for title/channel when DOM extraction fails.

## Architecture Patterns

### Toolbar ↔ MainWindowController delegate
ToolbarView owns UI; MainWindowController owns state. To add a toolbar action:
1. Add method to `ToolbarDelegate` protocol in `ToolbarView.swift`
2. Implement in `MainWindowController` (which conforms to the protocol)
3. For toolbar needing controller state (e.g. current channel), add query methods to the delegate protocol (e.g. `toolbarCurrentChannel`) — toolbar pulls, never stores app state.

### Per-channel speed pinning
- `Tab.currentChannel` — updated by `mediaBridge` JS messages
- `Tab.pinnedChannel` — set when user pins speed for a channel
- `Settings.channelSpeeds` — `[String: Float]` in UserDefaults, persists across sessions
- On navigation to a channel with saved speed, `MainWindowController` auto-applies it
- `updatePlaybackRate(_:pinned:)` shows 📌 indicator in rate field when pinned

### NSPopover for contextual actions
Use transient `NSPopover` with a simple `NSView` container + `PopoverActionButton` instances. Pattern in `ToolbarView.showSpeedPopover()`. Keep popovers minimal — 1-3 actions max. Popover closes itself via the action closure.

### App relaunch
`relaunchApp()` in MainWindowController: spawn `/bin/sh -c "sleep 0.5; open <bundlePath>"` then `NSApp.terminate`. Bound to Ctrl+Cmd+R.

## Workflow

- **Commit in logical chunks** as you go — don't wait until the end. Group related changes into a single commit with a descriptive message.
- **Relaunch shortcut**: Ctrl+Cmd+R — rebuilds are picked up by relaunching the app from `build/Debug/YTApp.app`.

## Style

- Minimal, modern — no heavy bezels, subtle hover states, smooth animations
- Dark backgrounds (black/near-black) for WebView container — no white flash
- NSTableView for reorderable lists; NSStackView for static layouts
- Custom NSView subclasses over standard AppKit controls when needed for polish

---
> Source: [Whoaa512/yt-app](https://github.com/Whoaa512/yt-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->

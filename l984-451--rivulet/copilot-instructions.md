## rivulet

> Rivulet is a tvOS media client for Plex and IPTV. It uses SwiftUI with MPV for video playback with HDR passthrough.

# Rivulet - Claude Context

Rivulet is a tvOS media client for Plex and IPTV. It uses SwiftUI with MPV for video playback with HDR passthrough.

## Quick Reference

- **Platform**: tvOS 26+ (Apple TV)
- **Language**: Swift 6
- **UI Framework**: SwiftUI
- **Video Player**: MPV via libmpv (Metal rendering, HDR passthrough)
- **Design Guide**: See `Docs/DESIGN_GUIDE.md` for UI/UX patterns

## Project Structure

```
Rivulet/
├── Models/
│   ├── Plex/           # Plex API models (PlexMetadata, PlexStream, etc.)
│   └── SwiftData/      # Persistent models (Channel, EPGProgram, PlexServer)
├── Services/
│   ├── Plex/           # PlexNetworkManager, PlexAuthManager, PlexDataStore
│   ├── Playback/       # MPVPlayerWrapper, AVPlayerWrapper, MediaTrack
│   ├── LiveTV/         # PlexLiveTVProvider, IPTVProvider, LiveTVDataStore
│   ├── IPTV/           # M3UParser, XMLTVParser, DispatcharrService
│   ├── Cache/          # CacheManager, ImageCacheManager
│   └── Focus/          # FocusMemory (tvOS section focus restoration)
├── Views/
│   ├── Player/         # UniversalPlayerView, PlayerControlsOverlay
│   │   ├── MPV/        # MPVPlayerView, MPVMetalViewController
│   │   └── PostVideo/  # Post-playback summary overlays
│   ├── Plex/           # PlexHomeView, PlexLibraryView, PlexDetailView
│   ├── LiveTV/         # ChannelListView, GuideLayoutView, LiveTVPlayerView
│   ├── Settings/       # SettingsView, SettingsComponents
│   ├── Components/     # CachedAsyncImage, GlassRowStyle
│   ├── TVNavigation/   # TVSidebarView, NavigationEnvironment
│   └── Root/           # SidebarView
└── Docs/
    └── DESIGN_GUIDE.md # Comprehensive UI/UX documentation
```

## Key Architectural Patterns

### Focus Management (tvOS)

Uses standard SwiftUI focus primitives with `FocusMemory` for section-level restoration. No custom focus scope manager — focus isolation is handled by system mechanisms:

- **`fullScreenCover`** — automatic focus isolation for overlays/popups
- **`TabView` with `sidebarAdaptable`** — system-managed sidebar/content focus
- **`@FocusState` + `.onAppear`** — setting initial focus in presented views
- **`FocusMemory`** — remembers and restores focus within scrollable sections

```swift
// Section focus memory
.focusSection()
.remembersFocus(key: "uniqueSectionKey", focusedId: $focusedItemId)

// Initial focus in fullScreenCover (no Namespace/resetFocus needed)
.onAppear {
    focusedUserId = profileManager.selectedUser?.id
}
```

### Video Player Architecture

- **UniversalPlayerView**: Main SwiftUI player container
- **UniversalPlayerViewModel**: Manages playback state, markers, post-video logic
- **MPVPlayerWrapper**: Bridges MPV to Swift (Combine publishers for state)
- **MPVMetalViewController**: UIViewController hosting Metal layer for MPV rendering

**Playback States**: `.idle`, `.loading`, `.playing`, `.paused`, `.buffering`, `.ended`, `.failed`

### Plex Metadata Hierarchy

For TV shows:
- **Show** (`grandparentRatingKey`) → **Season** (`parentRatingKey`) → **Episode** (`ratingKey`)
- Episode has `index` (episode number) and `parentIndex` (season number)

**Note**: Items from "Continue Watching" hub may lack parent metadata. Use `PlexNetworkManager.getMetadata()` to fetch full details.

### Glass UI Style

All focusable rows use consistent styling (see `Docs/DESIGN_GUIDE.md`):

```swift
// Background
.fill(isFocused ? .white.opacity(0.18) : .white.opacity(0.08))
.strokeBorder(isFocused ? .white.opacity(0.25) : .white.opacity(0.08), lineWidth: 1)

// Scale
.scaleEffect(isFocused ? 1.02 : 1.0)

// Animation
.animation(.spring(response: 0.3, dampingFraction: 0.7), value: isFocused)
```

## Common Tasks

### Presenting a Focus-Isolated Overlay

Use `fullScreenCover` — it provides automatic focus isolation without manual scope management:

```swift
.fullScreenCover(isPresented: $showOverlay) {
    MyOverlayView(isPresented: $showOverlay)
        .presentationBackground(.clear)  // See-through to content behind
}
```

In the presented view, use `@FocusState` with `.onAppear`:
```swift
@FocusState private var focusedItem: String?

.onAppear {
    focusedItem = defaultItemId
}
.onExitCommand {
    isPresented = false
}
```

### Fetching Next Episode

```swift
// Get episodes in current season
let episodes = try await networkManager.getChildren(
    serverURL: serverURL,
    authToken: authToken,
    ratingKey: metadata.parentRatingKey  // Season key
)

// Find next episode
let next = episodes.first(where: { $0.index == currentEpisodeIndex + 1 })
```

### Adding Settings

Use components from `SettingsComponents.swift`:
- `SettingsRow` - Navigation with chevron
- `SettingsToggleRow` - On/Off toggle
- `SettingsPickerRow` - Cycles through options
- `SettingsActionRow` - Action button (supports destructive)

### Image Loading

Always use `CachedAsyncImage` for remote images:
```swift
CachedAsyncImage(url: imageURL) { phase in
    switch phase {
    case .success(let image): image.resizable()
    case .empty: ProgressView()
    case .failure: Image(systemName: "photo")
    }
}
```

## Build & Run

```bash
# Build for tvOS Simulator
xcodebuild -scheme Rivulet -destination 'platform=tvOS Simulator,name=Apple TV' build

# Build for device
xcodebuild -scheme Rivulet -destination 'platform=tvOS,name=My Apple TV' build
```

## Key Files

| Purpose | File |
|---------|------|
| Main player | `Views/Player/UniversalPlayerView.swift` |
| Player state | `Views/Player/UniversalPlayerViewModel.swift` |
| MPV integration | `Services/Playback/MPVPlayerWrapper.swift` |
| Focus memory | `Services/Focus/FocusMemory.swift` |
| Plex API | `Services/Plex/PlexNetworkManager.swift` |
| Glass row styling | `Views/Components/GlassRowStyle.swift` |
| Settings components | `Views/Settings/SettingsComponents.swift` |
| Design patterns | `Docs/DESIGN_GUIDE.md` |

## Design Philosophy

From `Docs/DESIGN_GUIDE.md`:

- **Simplicity First**: Remove rather than add. The interface should feel calm.
- **Elegant Restraint**: Subtle effects (2% scale, soft glow) over flashy ones.
- **Liquid Glass**: Translucent backgrounds with subtle borders (tvOS 26 aesthetic).
- **Subtle Motion**: Small scale effects, natural animations.
- **Invisible Complexity**: Complex features should feel simple to use.

**Design Don'ts**:
- No over-decoration (gradients, unnecessary shadows)
- No aggressive animations (bouncing, overshooting)
- No redundant icons/labels
- No "just in case" features

## Troubleshooting

### Focus Not Working in Overlay
- Use `fullScreenCover` for focus-isolated overlays (provides its own focus hierarchy)
- Set initial focus via `@FocusState` in `.onAppear`
- Use `.onExitCommand` for Menu button dismissal
- For transparent overlays, add `.presentationBackground(.clear)` to the cover content

### Video Not Shrinking/Positioning
- Check `VideoFrameState` offset values (positive = padding from top-left with `.topLeading` anchor)
- Ensure `videoFrameState` is being set to `.shrunk`

### Post-Video Not Triggering
- Check if `hasTriggeredPostVideo` flag needs resetting
- Verify credits marker detection in `checkMarkers(at:)`
- Ensure `duration > 60` for time-based trigger (45s before end)

### Live TV Black Screen (Video Decodes But Doesn't Render)
- **tvOS has NO OpenGL** - only Metal (and Vulkan via MoltenVK)
- MPV must use `gpu-api=vulkan`, never `gpu-api=opengl`
- If you see "error setting option" followed by MoltenVK logs, check gpu-api setting
- Video decoding (VideoToolbox) can succeed while rendering fails due to misconfigured gpu-api

### Plex Live TV Not Starting (DVB Tuners)
- DVB tuners (TBS cards, etc.) don't have HDHomeRun stream URLs
- They require Plex server transcode via `/video/:/transcode/universal/start.m3u8`
- The transcode URL must include comprehensive client profile parameters
- Minimal URLs will cause "loading failed" - Plex needs to know client capabilities
- See `PlexLiveTVModels.buildPlexLiveTVStreamURL()` for required parameters

## MPV on tvOS

### Rendering Backends

| Mode | Video Output | GPU API | Use Case |
|------|-------------|---------|----------|
| VOD (HDR) | `gpu-next` | `vulkan` | Movies/TV with HDR passthrough |
| Live TV | `gpu` | `vulkan` | Live streams, lower resource usage |
| Simulator | `gpu` | `vulkan` | Testing (software decode) |

**Critical**: Never use `gpu-api=opengl` on tvOS - it doesn't exist and will fail silently.

### Live Stream Optimizations
```swift
// Minimal buffering for live TV
demuxer-max-bytes = 32MiB
demuxer-max-back-bytes = 0  // Can't seek anyway
cache-secs = 10

// Fast scaling (no HDR processing needed)
scale = bilinear
dscale = bilinear
dither = no
deband = no
```

## Plex Live TV

### Stream URL Types

| Tuner Type | URL Source | Notes |
|------------|-----------|-------|
| HDHomeRun | `PlexLiveTVChannel.streamURL` | Direct stream, works out of box |
| DVB (TBS, etc.) | Built via `buildPlexLiveTVStreamURL()` | Requires full transcode params |

### Required Transcode Parameters for DVB
```
X-Plex-Client-Profile-Name, X-Plex-Client-Profile-Extra
mediaIndex, partIndex, offset
container, segmentFormat, segmentContainer
videoCodec, videoResolution, maxVideoBitrate, videoQuality
audioCodec, audioBitrate, audioChannels
session (unique UUID per session)
```

Without these, Plex returns errors or empty responses → MPV reports "loading failed".

## Sentry Error Patterns

| Error | Likely Cause |
|-------|-------------|
| `MPV error: loading failed` | Bad stream URL, network issue, or missing transcode params |
| `MPV error: unrecognized file format` | Plex returned error page instead of stream |
| `HLS transcode session failed` | Incomplete transcode URL parameters |
| `HTTP 500 on /hubs` | Plex server issue (not client-side) |
| `NSURLErrorDomain -999 cancelled` | User navigated away, request timeout |

---
> Source: [l984-451/Rivulet](https://github.com/l984-451/Rivulet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->

## llizard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

llizardgui-host is a plugin-based GUI host application for the Spotify CarThing device. It uses raylib/raygui for graphics and provides an SDK that plugins use to draw UIs with a unified input/display abstraction.

**Target Platform:** ARM Linux (Spotify CarThing - Cortex-A7, ARMv7, hard float ABI)
**Graphics:** raylib with DRM backend for CarThing, standard raylib for desktop
**Display:** 800×480 logical resolution (SDK handles DRM rotation)

## Build Commands

### Desktop Build (Development)
```bash
mkdir -p build && cd build
cmake ..
make -j$(nproc)
```

### Cross-Compilation for CarThing (DRM)
```bash
mkdir -p build-armv7-drm && cd build-armv7-drm
cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain-armv7.cmake -DPLATFORM=DRM ..
make -j$(nproc)
```

### Build Outputs
- `build/llizardgui-host` - Desktop executable
- `build/nowplaying.so`, `build/redis_status.so` - Plugins copied to `plugins/` automatically
- `build-armv7-drm/llizardgui-host` - ARM binary for CarThing
- `build-armv7-drm/*.so` - ARM plugins

### Deployment
CarThing device: IP `172.16.42.2`, user `root`, password `llizardos`
```bash
scp build-armv7-drm/llizardgui-host root@172.16.42.2:/tmp/
scp build-armv7-drm/nowplaying.so root@172.16.42.2:/usr/lib/llizard/plugins/
```

## Architecture

### Host Application (`src/`)
- **main.c** - Plugin menu and main loop using SDK display/input functions
- **plugin_loader.c** - Dynamic loading of `.so` plugins from `./plugins/` directory via `dlopen`

### Plugin API (`include/llizard_plugin.h`)
Plugins export `LlzGetPlugin()` returning a `LlzPluginAPI` struct with callbacks:
- `init(width, height)` - Called when plugin is selected
- `update(input, deltaTime)` - Per-frame update with input state
- `draw()` - Render plugin content
- `shutdown()` - Cleanup when exiting plugin
- `wants_close()` - Optional signal to return to menu

### SDK (`sdk/`)
Abstracts raylib/DRM platform differences:
- **llz_sdk_display.h** - `LlzDisplayInit/Begin/End/Shutdown` for render target management
- **llz_sdk_input.h** - `LlzInputState` with buttons, touch, gestures, scroll wheel
- **llz_sdk_layout.h** - Helper functions for layout math
- **llz_sdk_media.h** - Redis-backed media state access and playback commands
- **llz_sdk_image.h** - Blur effects and cover/contain image scaling
- **llz_sdk_config.h** - Global and per-plugin configuration
- **llz_sdk_background.h** - 9 animated background styles
- **llz_sdk_subscribe.h** - Event callbacks for media changes
- **llz_sdk_navigation.h** - Plugin-to-plugin navigation requests
- **llz_sdk_font.h** - Centralized font loading with automatic path resolution
- **llz_sdk_connections.h** - Service connection status (Spotify auth, etc.)

Plugins include `llz_sdk.h` and receive `const LlzInputState*` in their update callback.

### Fonts (`fonts/`)
Centralized font storage for consistent text rendering:
- **ZegoeUI-U.ttf** - Primary UI font (CarThing system font)
- Copy fonts from CarThing: `./scripts/copy-carthing-fonts.sh`
- SDK searches: `/var/local/fonts/`, `/tmp/fonts/`, `./fonts/`, system fonts
- See `fonts/README.md` for detailed font API usage

### Shared Libraries (`shared/`)

**Notification System** (`shared/notifications/`)
- Reusable popup notifications that plugins can opt-in to use
- Three styles: Banner (top/bottom bar), Toast (corner popup), Dialog (modal with buttons)
- Queue system for sequential display
- Callbacks for tap, dismiss, timeout, button press
- Built-in plugin navigation on tap
- Usage: `#include "llz_notification.h"` and link with `llz_notifications`

### Available Plugins

**nowplaying** (`plugins_src/nowplaying/`)
- Music player UI with multiple display modes (ADVANCED, NORMAL, BAREBONES, ALBUM_ART)
- Theme system with multiple color variants
- Clock overlay, progress scrubbing, playback controls
- Modular structure: `core/`, `widgets/`, `screens/`, `overlays/`

**redis_status** (`plugins_src/redis_status/`)
- Displays live Redis/MediaDash state from the golang_ble_client daemon
- Two-column layout: Connection status (left), Now Playing media (right)
- Header with Redis/BLE connection indicators
- Requires Redis running on CarThing (`sv start redis`)
- Uses SDK's `LlzMedia*` APIs to fetch state from Redis

**spotify** (`plugins_src/spotify/`)
- Full-featured Spotify control interface leveraging Janus Android companion app
- Three screens: Hub (now playing), Queue (browser with skip-to), Controls (shuffle/repeat/like/volume)
- Spotify-themed color palette (green accent on dark background)
- Album art display with automatic loading
- Connection status indicator for Spotify auth
- Swipe/scroll navigation between screens
- Track change detection with automatic queue refresh
- **Shuffle/Repeat/Like controls** via Spotify Web API:
  - Toggle shuffle on/off
  - Cycle repeat modes (off → track → context → off)
  - Like/unlike current track (saves to Spotify library)
- Requires Spotify OAuth authentication in Janus companion app

**albums** (`plugins_src/albums/`)
- Browse saved Spotify albums in a horizontal carousel
- Large album art cards with smooth spring-based scrolling
- Displays album name, artist, year, and track count
- Album art loaded from preview cache (150x150) or full cache (250x250)
- Select album to play and navigate to Now Playing
- Requires Spotify OAuth authentication in Janus companion app

**artists** (`plugins_src/artists/`)
- Browse followed Spotify artists in a horizontal carousel
- Circular artist cards for profile aesthetic
- Displays artist name, primary genre, and follower count
- Smooth spring-based scrolling physics
- Artist art loaded from cache with BLE request fallback
- Select artist to play (shuffles top tracks) and navigate to Now Playing
- Uses cursor-based pagination (Spotify artists API requirement)
- Requires Spotify OAuth authentication in Janus companion app

### Plugin Structure (nowplaying example)
```
plugins_src/nowplaying/
├── nowplaying_plugin.c     # Plugin entry point with LlzGetPlugin()
├── core/                   # Theme system, effects
├── widgets/                # Reusable UI components
├── screens/                # Screen implementations
└── overlays/               # Overlay system (clock, etc.)
```

## Key Patterns

### Input Handling
Use `LlzInputState` fields, not raw raylib calls, for CarThing hardware compatibility:
- `backPressed`, `selectPressed`, `upPressed`, `downPressed` - Hardware buttons
- `screenshotPressed` - Screenshot button (CT_BUTTON_SCREENSHOT)
- `scrollDelta` - Scroll wheel/rotary encoder
- `tap`, `doubleTap`, `hold`, `swipeLeft/Right/Up/Down` - Touch gestures

### Platform-Specific Code
```c
#ifdef PLATFORM_DRM
    // CarThing-specific behavior
#else
    // Desktop behavior
#endif
```

### Font Loading
Use SDK font loader for consistent text rendering across platforms:
```c
#include "llz_sdk.h"

// Get default UI font
Font font = LlzFontGetDefault();

// Get font at specific size
Font titleFont = LlzFontGet(LLZ_FONT_UI, 36);

// Draw text helpers
LlzDrawText("Hello", 100, 100, 24, WHITE);
LlzDrawTextCentered("Title", 400, 50, 32, GOLD);
LlzDrawTextShadow("Shadow", 100, 200, 28, WHITE, BLACK);

// Measure text
int width = LlzMeasureText("Text", 24);
```

### Connections API
Check service connection status (Spotify, etc.) via the Android companion app:
```c
#include "llz_sdk.h"

// Initialize connections module (auto-checks every 3 minutes)
LlzConnectionsInit(NULL);

// In plugin update loop
void PluginUpdate(const LlzInputState *input, float dt) {
    LlzConnectionsUpdate(dt);  // Handle auto-refresh timing

    // Check if Spotify is connected
    if (LlzConnectionsIsConnected(LLZ_SERVICE_SPOTIFY)) {
        DrawText("Spotify: Connected", 100, 100, 20, GREEN);
    }

    // Manual refresh on button press
    if (input->selectPressed) {
        LlzConnectionsRefresh();
    }
}

// Get detailed status
LlzServiceStatus status;
if (LlzConnectionsGetServiceStatus(LLZ_SERVICE_SPOTIFY, &status)) {
    // status.state: LLZ_CONN_STATUS_CONNECTED, DISCONNECTED, ERROR, CHECKING, UNKNOWN
    // status.error: error message if state is ERROR
    // status.lastChecked: Unix timestamp of last check
}
```

### Spotify Playback Controls
Control Spotify playback features via the SDK (requires Spotify OAuth in Janus app):
```c
#include "llz_sdk.h"

// Shuffle control
LlzMediaSetShuffle(true);   // Enable shuffle
LlzMediaSetShuffle(false);  // Disable shuffle
LlzMediaToggleShuffle();    // Toggle current state

// Repeat control
LlzMediaSetRepeat(LLZ_REPEAT_OFF);     // Disable repeat
LlzMediaSetRepeat(LLZ_REPEAT_TRACK);   // Repeat current track
LlzMediaSetRepeat(LLZ_REPEAT_CONTEXT); // Repeat playlist/album
LlzMediaCycleRepeat();                 // Cycle: off → track → context → off

// Like/Unlike tracks (save to Spotify library)
LlzMediaLikeTrack(NULL);     // Like current track
LlzMediaUnlikeTrack(NULL);   // Unlike current track
LlzMediaLikeTrack("trackId"); // Like specific track by Spotify ID

// Request fresh state from Spotify API
LlzMediaRequestSpotifyState();

// State is available in LlzMediaState:
LlzMediaState state;
if (LlzMediaGetState(&state)) {
    bool shuffle = state.shuffleEnabled;
    LlzRepeatMode repeat = state.repeatMode;  // LLZ_REPEAT_OFF, TRACK, CONTEXT
    bool liked = state.isLiked;
    const char *trackId = state.spotifyTrackId;
}
```

### Adding a New Plugin
1. Create `plugins_src/yourplugin/yourplugin_plugin.c`
2. Implement `LlzGetPlugin()` returning a `LlzPluginAPI*`
3. Add sources and target to `CMakeLists.txt` (see `nowplaying_plugin` example)
4. Plugin `.so` is auto-copied to `plugins/` directory post-build
5. Optional: Add `llz_notifications` to link libraries for popup notifications
6. Optional: Create `supporting_projects/salamanders/yourplugin/` for plugin resources (question banks, data files, scripts)

## External Dependencies

- **raylib** - Graphics library (in `external/raylib`)
- **raygui** - Immediate-mode GUI (in `external/raygui`)
- **hiredis** - Redis C client (in `external/hiredis`) - used by redis_status plugin

## Internal Libraries

- **llz_sdk** - Core SDK with display, input, media, config, background, subscribe, navigation, font
- **llz_notifications** - Popup notification system (in `shared/notifications/`) - opt-in for plugins

## Scripts

- **scripts/copy-carthing-fonts.sh** - Copy fonts from connected CarThing device to `fonts/`

## CarThing Redis Setup

The redis_status plugin requires Redis running on the CarThing:
```bash
# Check status
sshpass -p llizardos ssh root@172.16.42.2 "sv status redis"

# Start Redis
sshpass -p llizardos ssh root@172.16.42.2 "sv start redis"

# Test connection
sshpass -p llizardos ssh root@172.16.42.2 "redis-cli ping"
```

Redis is populated by the `golang_ble_client` daemon which bridges BLE media data to Redis keys.

## Supporting Projects

### `supporting_projects/llizard_firmware`
Custom firmware build system and flashing tools for CarThing.

**Key Components:**
- `tools/flasher/` - Native raylib GUI flasher for flashing CarThing
- `tools/superbird-tool/` - Python reference flasher (from thinglabsoss)
- `scripts/` - Firmware build pipeline (5-stage Docker build)
- `output/` - Built firmware images (partition files)

**GUI Flasher (`tools/flasher/`)**
Native C/raylib application implementing Amlogic USB boot protocol:
- `src/amlogic_protocol.c` - USB protocol (BL2 boot, AMLC, partition writes)
- `src/flash_protocol.c` - High-level flashing workflow
- `src/main.c` - raylib GUI

**Build & Run:**
```bash
cd supporting_projects/llizard_firmware/tools/flasher
mkdir build && cd build
cmake .. && make -j$(nproc)
sudo ./llizard_flasher
```

**Protocol Details:**
- USB VID:PID `0x1b8e:0xc003` (Boot ROM/Burn mode)
- BL2 loaded to `0xfffa0000`, executed via control transfer `0x05`
- AMLC protocol uploads bootloader in chunks with checksum
- Partition writes use `amlmmc write` commands via USB bulk transfers
- Bootloader writes at 512-byte offset, always timeout (expected)

See `supporting_projects/llizard_firmware/README.md` for full protocol documentation.

### `supporting_projects/carthing_installer`
SSH-based installer for development (doesn't reflash, just copies files over SSH).

**Build & Run:**
```bash
cd supporting_projects/carthing_installer
mkdir build && cd build
cmake .. && make
./carthing_installer
```

### `supporting_projects/salamander`
Desktop raylib application for managing plugin installation on CarThing via SSH/SCP.

**Features:**
- Three-section UI: Device Only, Synced, Local Only
- Drag-and-drop plugin install/uninstall
- Fire salamander visual theme

**Build & Run:**
```bash
cd supporting_projects/salamander
mkdir build && cd build
cmake .. && make -j$(nproc)
./salamander
```

### `supporting_projects/salamanders`
Per-plugin supporting files directory. Contains one subfolder per plugin for resources that aren't source code.

**Structure:**
```
salamanders/
├── flashcards/
│   ├── questions/           # Runtime question banks (copied to plugins/ at build)
│   ├── scraped_questions/   # Raw OpenTDB scraped data
│   ├── legacy_questions/    # Old question files
│   └── scrape_opentdb.py    # Question scraper utility
├── millionaire/
│   └── questions/           # Runtime question bank
├── nowplaying/              # Now Playing resources
└── ...                      # One folder per plugin
```

**Build Integration:** CMakeLists.txt copies `salamanders/{plugin}/questions/` → `plugins/{plugin}/questions/` at build time.

### `supporting_projects/golang_ble_client`
- Sourced from `llizardgui-raygui/supporting_projects` and mirrored here for reference/integration work.
- Always-on Go daemon that bridges the Android MediaDash phone and the llizard UI: it speaks BLE on one side and exposes all shared state/commands via Redis on the other.
- **Data Flow**
  - *BLE → Redis*: Metadata, playback progress, connection metrics, and album art notifications are decoded inside `internal/ble`/`internal/albumart` and written into Redis hashes/lists such as `media:track`, `media:progress`, `system:ble_connected`, plus cached blobs under `/var/mediadash/album_art_cache/` with mirror keys (`media:album_art_cache:<hash>`).
  - *Redis → BLE*: UI drops commands (`play`, `pause`, `seek`, `skip`, `volume`) into queues like `system:playback_cmd_q`. `internal/redis` dequeues, validates, and `internal/ble` forwards them to the BLE control characteristic with retry/backoff.
- **Key Packages**
  - `cmd/mediadash-client`: entry point that loads `config.json`, wires goroutines (BLE IO, Redis pump, status writers).
  - `internal/ble`: scanning/pairing, ATT IO, connection metrics, album-art chunking.
  - `internal/redis`: typed helpers (`StoreMediaState`, `StoreConnectionHealth`, `QueuePlaybackCommand`, etc.) that encapsulate the Redis schema.
  - `internal/albumart`: normalizes hashes, persists blobs to `/var/mediadash/album_art_cache`, and tracks cache state in Redis.
- **Documentation & Tooling**: `LLM_OVERVIEW.md`, `COMMAND_PROCESSING.md`, `ALBUM_ART_PROTOCOL_FIXES.md`, and `PERFORMANCE_OPTIMIZATIONS.md` describe the BLE protocol and Redis contract; `build-deploy.sh`, `test_build.sh`, and the Dockerfile provide build/deploy workflows.
- Use this project as the canonical definition for the SDK's upcoming Redis getter/setter APIs (track metadata, progress, connection status, command queues).

### `supporting_projects/llizardOS`
Firmware build system for CarThing. Produces flashable Void Linux images configured to run llizardgui-host natively on DRM with Redis-backed BLE media bridging.

**Key Components:**
- `build.sh` - Main build script running 5-stage pipeline
- `Dockerfile` - Docker build environment with all dependencies
- `scripts/stages/` - Build stages (00: rootfs, 10: system config, 20: llizard stack, 30: cleanup, 40: image)
- `scripts/services/` - runit service definitions (redis, mediadash-client, llizardgui, bluetooth)
- `resources/bins/` - Pre-built ARM binaries (llizardgui-host, mediadash-client, plugins)
- `resources/fonts/` - TTF fonts for llizardgui-host
- `resources/stock-files/` - CarThing stock libraries (libMali)

**Build Prerequisites:**
```bash
cd supporting_projects/llizardOS
# 1. Download stock files
cd resources/stock-files && ./download.sh && cd ../..
# 2. Copy pre-built ARM binaries
cp ../../../build-armv7-drm/llizardgui-host resources/bins/
cp ../../../build-armv7-drm/*.so resources/bins/plugins/
cp ../golang_ble_client/mediadash-client resources/bins/
# 3. Copy fonts
cp ../../../fonts/*.ttf resources/fonts/
```

**Build:**
```bash
just docker-qemu  # Setup ARM emulation (non-ARM hosts)
just docker-run   # Build image
```

**Output:** `output/nocturne_image_vX.X.X.zip` - Flashable via Terbium

**Runtime Stack:**
- `llizardgui` - Native raylib GUI on DRM (no Weston/Chromium)
- `mediadash-client` - BLE media bridge (golang_ble_client)
- `redis` - Data store for media state

See `supporting_projects/llizardOS/CLAUDE.md` for full documentation.

### `supporting_projects/mediadash-android` (Janus)
Android companion app that provides media metadata over BLE to the CarThing.

**Key Features:**
- BLE GATT server exposing media state to CarThing
- Universal media control via NotificationListenerService
- Built-in podcast player with subscription management
- Album art transfer with optimized chunking
- Synced lyrics from LRCLIB API
- Media channel selection (Spotify, YouTube Music, Podcasts)
- **Spotify OAuth integration** via spotSDK (page 4 in app)

**Spotify Integration:**
- Uses spotSDK for OAuth 2.0 PKCE authentication
- Accessible via 4th page in the horizontal pager
- Stores tokens securely in DataStore
- Redirect URI: `janus://spotify-callback`
- Requires Spotify Developer Dashboard app configuration

**Build:**
```bash
cd supporting_projects/mediadash-android
./gradlew assembleDebug
./gradlew installDebug
```

See `supporting_projects/mediadash-android/README.md` for full BLE protocol documentation.

### `supporting_projects/spotSDK`
Android library for Spotify Web API integration with OAuth 2.0 PKCE authentication.

**Features:**
- OAuth 2.0 PKCE authentication (no client secret required)
- Recently played tracks, top artists/tracks
- Saved library, playlists, followed artists
- Built-in caching with configurable durations
- Rate limit handling with automatic retry
- UI components: SpotifyLoginButton, SpotifyUserIndicator

**Usage:**
```kotlin
// Configure in Application.onCreate()
SpotSDK.configure(context) {
    clientId = "your_spotify_client_id"
    redirectUri = "yourapp://spotify-callback"
}

// Use the SDK
val sdk = SpotSDK.getInstance()
sdk.getRecentTracks { result ->
    when (result) {
        is SpotifyResult.Success -> handleTracks(result.data)
        is SpotifyResult.Error -> handleError(result)
    }
}
```

**Limitations:**
- Development mode: max 25 authenticated users
- Extended quota requires Spotify review
- Some endpoints restricted (recommendations, audio features)

See `supporting_projects/spotSDK/README.md` for full documentation.

### `supporting_projects/townhaus`
Personal portfolio and creative showcase website hosted at pautown.github.io. An interactive web art space featuring a generative grid interface with contemplative design.

**Key Features:**
- Draggable fragment tiles with localStorage persistence
- Atmospheric effects (particles, mist, grain texture, stillness-triggered vines)
- Keyboard shortcuts: `[b]` breathing, `[d]` dark mode, `[s]` scatter
- Project showcase pages including llizardOS

**Structure:**
- `index.html` - Main landing page with fragment grid
- `shared.css` - Shared aesthetic system
- `pages/` - Individual fragment pages (silence, orb, door, etc.)
- `pages/projects/llizardos.html` - llizardOS project showcase
- `docs/vibe.md` - Design philosophy guide

**Development:**
```bash
cd supporting_projects/townhaus
python3 -m http.server 8000
# Open http://localhost:8000
```

See `supporting_projects/townhaus/CLAUDE.md` for full documentation.

---
> Source: [pautown/llizard](https://github.com/pautown/llizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->

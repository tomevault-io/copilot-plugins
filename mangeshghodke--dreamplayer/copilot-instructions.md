## dreamplayer

> A cross-platform video player built with Flutter.

# DreamPlayer

A cross-platform video player built with Flutter.

## Goal

A video player app supporting:
- **Android** (primary, tested on user's Android phone — CPH2573, Android 16) and **iOS/iPad** (user's iPad Pro M2)
- **All audio codecs**: DTS, DTS-HD, E-AC3, AC3, TrueHD, etc.
- **Dolby Vision** where the display supports it
- **FFmpeg-based** decoding engine

## Current status

- App **UI skeleton** done (library, player, settings; dark theme).
- **HDR / codec on-screen display** done (Dolby Vision, HDR10+, HDR10, SDR; E-AC3, DTS-HD, TrueHD, AAC, ...).
- **Responsive layout** — no overflow on phones/tablets/landscape/large text.
- **Native refresh rate** selected at startup (verified 120 Hz on device).
- **DOLBY VISION PLAYBACK WORKS on Android via ExoPlayer/Media3 PlatformView.**
  Verified on-device: the DV P8 test file (`dolby-vision-people`) decodes on the
  Qualcomm hardware **`c2.qti.dv.decoder`** at 4K 3840x2160@60 fps with zero
  dropped frames, correct colors (no mpv pink/green), audio via
  `c2.dolby.eac3.decoder` / Media3 `FFmpegAudioRenderer`. Implementation:
  native `SurfaceView` PlayerView in a Flutter `AndroidView` (hybrid-composition
  fallback keeps its own SurfaceFlinger layer → real HDR to the display) +
  `MethodChannel`/`EventChannel` per view. `ExoPlayerController.open()` issued
  before the platform view attaches is queued and flushed in `_attach`.
  **Gotcha fixed:** the backend must `setState` after creating the controller,
  or the buttons/video layer stay frozen in the pre-init state.
- **iOS/iPad playback via AetherEngine (2026-08)** — the raw **AVPlayer**
  platform view was swapped for an **AetherEngine**-backed one
  (`ios/Runner/AvPlayerView.swift`, `UiKitView` on the Dart side) behind the
  exact same `dreamplayer/exo_<id>` method/event channel contract, so the Dart
  `ExoPlayerController` is unchanged. AetherEngine adds what AVPlayer alone
  cannot: **FFmpeg demux of MKV/TS/AVI/WebM**, **DTS/DTS-HD/TrueHD/E-AC3 audio**
  (AudioToolbox + libavcodec), **Dolby Vision / HDR10(+) via the native AVPlayer
  path** for Apple containers. `engine.bind(view:)` mounts `AetherPlayerView`
  (own `AVPlayerLayer` → real HDR where the panel supports it; iPad Pro M2
  does). Engine added as an SPM dependency (`project.pbxproj`, pinned
  `upToNextMajorVersion` from 6.21.0) — Xcode auto-resolves FFmpegBuild's
  dynamic FFmpeg xcframeworks into the app bundle. **CI-green** (run on commit
  `82b3dd9`). **Verified on-device (2026-08):** local/Documents files AND SMB streams
  play on the iPad Pro M2 via the former in-app SMB browser (AMSMB2) — since
  hidden from the UI (2026-08-13, see "SMB / network shares"), NAS playback is
  via CX/Files "Open with".
  **Minimum iOS 18.0** (`IPHONEOS_DEPLOYMENT_TARGET = 18.0`; builds for
  iOS 18 through the latest, iPhone and iPad).
  - Channel mapping: state 1/2/3/4 (idle/buffering/ready/ended); DV surfaces as
    `dvhe.<profile>.06` so Dart's `dv`-prefix detection fires; `colorTransfer`
    6 for HDR10/10+/DV, 7 for HLG. Audio/subtitle tracks pushed via
    `currentTracks`; `selectAudioTrack`/`selectSubtitleTrack`/`clearSubtitle`
    mapped 1:1 to engine calls.
  - **SMB audio-track switch fix (2026-08)**: `selectAudioTrack` on an SMB
    stream used to fail with "Demuxer: open failed (Operation not permitted
    (-1))". AetherEngine's reload reuses the RETAINED custom `SMBIOReader`
    (`keepCustomReader: true`); the old session teardown calls
    `SMBIOReader.cancel()` which marks the CURRENT in-flight read cancelled —
    when it lands on the new probe's first read, that read aborts with -1
    (EPERM). `AvPlayerView.setAudioTrack` now detects SMB streams
    (`isSMBStream`/`smbToken`) and calls `reopenSMBStream(audioIndex:)`
    instead: `SMBClient.reconnect(for:)` mints a FRESH `SMBConnection` on the
    same server/share/path (swapping it into the registry, returning the
    displaced stale connection), then `engine.load(source:startPosition:
    options:audioSourceStreamIndex:)` rebuilds at the current playhead with the
    requested track. The stale connection is closed only AFTER the engine
    finishes swapping readers, so the running session is never interrupted
    mid-teardown. All SMB readers use `ownsSource: false` — SMBClient owns
    connection lifetime (closed on `closeShare`).
  - **SMB buffering / read-ahead fix (2026-08)**: while SMB audio switching was
    fixed, SMB streams still hit the buffering spinner mid-playback. Root cause:
    `SMBIOReader` bridges every FFmpeg read to a SYNCHRONOUS SMB round-trip
    (256 KB per read, zero prefetch), so a Wi-Fi latency spike starves the
    engine's loopback HLS producer and AVPlayer drops into
    `waitingToPlayAtSpecifiedRate. `AvPlayerView` now loads all SMB streams
    through a new `BufferedSMBReader` (`ios/Runner/BufferedSMBReader.swift`), a
    read-ahead sliding-window `IOReader` (32 MiB window / 4 MiB chunks fetched
    on a background `Task.detached` prefetch): demux reads are served from
    memory while the next chunk streams from the NAS, so latency only bites on
    a seek or a full window drain. Same idea as Nova's 48 MB ring buffer. It
    keeps `SMBIOReader`'s lifecycle contract (`cancel()` only bumps a cancel
    epoch so teardown unblocks THIS read without poisoning a reload reopen;
    `close()` stops prefetch but does NOT close the connection — SMBClient owns
    it; `makeIndependentReader` clones share the transport, which SMBClient
    serialises internally). Prefetch is error-recoverable: a failed fetch
    parks the task and a later read/seek kick (re-anchor resets `fetchFailed`)
    retries instead of dying permanently. Used for both the initial `open()`
    and `reopenSMBStream(audioIndex:)` (both SMB source sites). CI will verify
    the iOS build; on-device buffering was still to be re-measured.
  - **Replay / scrub-after-end**: AetherEngine's `.ended` is terminal (seek and
    play are explicit no-ops there), so `AvPlayerView` keeps the last-opened
    `url` + `LoadOptions` and a `play`/`seekTo` arriving in `.ended` reloads the
    session (`reloadSession(at:)` — start for replay, target for scrubber
    pull-back) instead of calling a no-op seek. The active subtitle track is
    re-applied after the reload.
  - **Subtitles render host-side**: AetherEngine decodes cues into
    `engine.$subtitleCues` and its `AetherPlayerView` does NOT paint them, so
    `AvPlayerView` draws its own `SubtitleOverlayView` (text + PGS/DVB bitmap
    cues positioned against the aspect-fit video rect; `zPosition = 1000` above
    the re-attached video layer). Sibling sidecar files (SRT/ASS/VTT) auto-pair
    as `ExternalSubtitleTrack`s (best filename match `isDefault`, id =
    `externalSubtitleTrackIDBase` + ordinal) — like Android.
  - A Documents-folder file browser (`ios/Runner/FileBrowser.swift`, same
    `dreamplayer/files` contract) plus
    `UIFileSharingEnabled`/`LSSupportsOpeningDocumentsInPlace` mean videos are
    dropped into the app via the Files app ("On My iPad → DreamPlayer") and
    played in-app. **iOS "Open with" works too** — `CFBundleDocumentTypes`
    (system video UTIs) + **`UTImportedTypeDeclarations`** (custom UTIs mapping
    `mkv`/`ts`/`m2ts`/`webm`/`wmv`/`flv`/`ogv`/`rmvb`/`mpg`/`vob`… to
    `public.movie`, since iOS has no system UTI for those containers) put
    DreamPlayer in the Files/share sheet for every container, and
    `ios/Runner/IntentBridge.swift` mirrors the Android `dreamplayer/intent`
    contract (`getInitialIntent` on launch via scene connection options /
    launch options; `open` from `application(_:open:options:)` +
    `scene(_:openURLContexts:)`, deduped). Security-scoped file URLs from the
    Files app keep their access scope for the playback session. Opening a file
    auto-plays it: the intent pushes `PlayerScreen`, whose `open()` runs with
    `autoplay: true`.
- **media_kit / libmpv fully REMOVED** from `pubspec.yaml`, `main.dart`,
  `player_screen.dart`, and the APK (no more `libmpv.so`/mediakit libs; only
  `libflutter.so` + `libmedia3ext.so` remain).
- **Subtitles done (embedded + sideloaded)**: every sibling subtitle file in
  the video's folder auto-attaches (SRT, SSA/ASS, WebVTT, TTML, SAMI, MicroDVD,
  MPL2, SubViewer via custom parsers), the best match auto-selects, and the CC
  button opens a full track picker over embedded + sideloaded tracks.
- **New direction**: playback on Android via **ExoPlayer/Media3** in a Flutter
  **PlatformView + MethodChannel** (HDR/DV-capable native surface), modeled on
  **Nova Video Player** architecture. Keep the Flutter UI/shell, the rendering/
  decoding layer is native Android code.

## Tech stack

| Concern | Choice | Notes |
|---|---|---|
| Framework | Flutter (stable, 3.44.x) | Cross-platform, single codebase |
| Playback engine (Android) | **ExoPlayer / Media3** (native, in PlatformView) | HDR/DV passthrough-capable; working (`c2.qti.dv.decoder`). |
| Playback engine (iOS/iPad) | **AetherEngine** (native, in PlatformView) | `AvPlayerView.swift` + `AetherEngine` SPM dep; FFmpeg demux/decode + native AVPlayer path for DV/HDR; cues drawn by host `SubtitleOverlayView`. |
| SMB client (iPad) | **AMSMB2** (browse) + **AetherEngineSMB** (playback) | `SMBClient.swift` (channel `dreamplayer/smb`); AMSMB2 for browsing, `AetherEngineSMB` `SMBConnection`/`SMBIOReader` custom source for playback. |
| Android audio decode | Media3 `FFmpegAudioRenderer` (ffmpeg extension) | DTS, DTS-HD, E-AC3, AC3, TrueHD — same bundled-FFmpeg approach Nova uses. |
| Reference architecture | **Nova Video Player** (`nova-video-player/aos-AVP`) | See "Playback research notes". |
| ~~media_kit / libmpv~~ | **retired** | Cannot do Dolby Vision (no passthrough, no RPU). |
| Permissions | `permission_handler` | Runtime `READ_MEDIA_VIDEO` request on video open |
| Refresh rate | `flutter_displaymode` | Selects highest refresh mode at startup |

### Device research notes (user's Android phone)
- Display: 1440x3168, supports 60/90/120 Hz, max luminance ~1400 nits.
- `supportedHdrTypes=[1, 2, 3, 4]` → **HDR10, HDR10+, Dolby Vision, HLG** all supported on the panel. Good news for the Dolby Vision goal.
- The phone runs at 60 Hz when the UI is idle and jumps to 120 Hz during animations (adaptive). Verified via `dumpsys SurfaceFlinger` after app launch.

### Playback research notes
- **`media_kit`/mpv is dead for this project's DV goal.** On-device verification:
  - HDR10 (PQ/BT.2020) tone-maps to SDR correctly via mpv `gpu` vo.
  - Dolby Vision P8 renders **pink/green**: mpv v0.36 + FFmpeg 6.0 cannot parse the
    DOVI RPU (file VUI reports `color_transfer/primaries=unknown`), so wrong colors.
    `gpu-next` vo is a frozen frame (media_kit renders via legacy `gpu` path; mpv
    PR #16818 pending). `hwdec:no` (software) gives correct colors but is too slow
    for 4K.
  - Flutter textures (what media_kit uses) have **no HDR path on any platform**
    (media-kit issue #615) — the display only ever sees SDR.
- **New plan (ExoPlayer/Media3, Nova-style):**
  - Render video into a native Android `SurfaceView`/`SurfaceFlinger`-driven
    `PlatformView` so the display receives real HDR/DV signal (the panel supports
    DV — `supportedHdrTypes` includes it).
  - **Nova Video Player architecture** (`https://github.com/nova-video-player/aos-AVP`):
    entry-point repo with `default.xml` manifest. Sub-repos:
    - `aos-Video` — Video UI (Kotlin, ExoPlayer-based playback)
    - `aos-MediaLib` — media library / MediaStore scanning
    - `aos-FileCoreLibrary` — file management (root/network)
    - `aos-avos` — C core multimedia engine using FFmpeg (probing/decoding)
    - Uses ExoPlayer (`exoplayer.xml`) + FFmpeg audio extension for the lossless
      codecs. Building: `cd Video && ./gradlew -Puniversal assembleNoamazonRelease`
  - Android audio codecs map to Media3 `FFmpegAudioRenderer` extension modules;
    `dts`, `truehd`, `eac3`, `ac3` etc. are FFmpeg decoders.
- **Nova buffering / read-ahead (how Nova smooths slow SMB/Wi-Fi; source = `aos-avos`)**:
  - **48 MB ring buffer for network streams** — `Source/avos_mp_video.c:256`
    `stream_set_buffer_size(video->s, 48)`. Wiki "Buffering" history: 12→24 MB
    (2015, high-bitrate 4K) → 48 MB (2022, 2× speed). "Used as cache before the
    parser to tackle buffering issues." Local default is `STREAM_DEFAULT_BUFFER_SIZE`
    64 MB / `STREAM_LARGE_BUFFER_SIZE` 128 MB (`Include/stream.h:41-42`).
  - **Ring buffer + dedicated background pthread** — `Source/stream_buffer.c:162`
    `_buffer_thread` loops `pthread_mutex_trylock` → `buffer->buffer(buffer,1)`,
    sleeps 500 ms when full (`BUFFER_SLEEP`). It refills when the parsed-ahead
    media drops below `stream_drive_wake_sleep = 5000` (5 s, `stream_buffer.c:37`);
    when actively playing it uses `stream_drive_wake_no_sleep = 2000` (s) — i.e.
    keep the ring essentially **always full**.
  - **Rate-aware refill threshold** — `_calc_buffer_threshold` (`stream_buffer.c:55`)
    predicts seconds-ahead from the measured `vcurrent_rate`/`acurrent_rate`
    (min rate floor 250 kbit/s), not just free space. This informed the in-app
    SMB read-ahead design (see "SMB / network shares" roadmap section).
  - **Debugging**: `av.sh smb` prints the current max buffer size; `av.sh dbgv 2`
    shows fill rate.
  - **SMB library**: Nova's SMBv2/3 support is via **jcifs-ng** (wiki "SMBv2 3",
    Apr 2020; earlier jcifs 1.3.19 was SMBv1-only) — **NOT smbj** (see Libraries
    table correction). Nova's C core has no SMB IO module (`stream_io_*.c` are all
    local); network files are opened by the Android app layer and fed to the engine.
  - iOS/iPad DV is restricted by Apple APIs — ExoPlayer/Media3 is Android-only;
    iOS will need a separate native path (AVPlayer). For now focus Android.

## Implemented features

- **HDR detection** (`lib/models/hdr_format.dart`, `lib/utils/codec_info.dart`): parses hints like `DV P8`, `HDR10+`, `HDR10` into a `HdrFormat` (incl. `HLG`); maps raw codec names (`dts_hd`, `eac3`, `truehd`, `aac`, ...) to display labels. Live detection from Media3 format info: DV track codecs (`dvhe`/`dvh1`/`dvav`), `colorTransfer` (6→HDR10, 7→HLG).
- **Real playback** (`lib/screens/player_screen.dart`): Android uses a native **ExoPlayer/Media3 PlatformView** (`lib/services/exo_player.dart`) with live codec/HDR/resolution chips, play/pause, seek, ±10s, mute, fullscreen, buffering spinner, error surface. Non-Android shows a "not yet supported" message. Widget tests run playback-less (`FLUTTER_TEST` gate).
- **Android permissions**: `READ_MEDIA_VIDEO` (+ `READ_EXTERNAL_STORAGE` ≤ API 32) requested at runtime via `permission_handler` when a video is opened. `compileSdk = 37` required by `permission_handler`.
- **Player overlay** shows HDR format + video/audio codec + resolution chips; library cards show an HDR badge + audio codec label.
  - **DV dedup**: for Dolby Vision the purple HDR chip already says "Dolby Vision", so the redundant video-codec chip is suppressed (no "Dolby Vision" twice).
  - **Chip layout**: landscape puts back button + title + chips in one `Wrap` on the same row; portrait shows title row, then chips `Wrap` below.
- **Player controls**: top bar (back + title) and a slim bottom bar (time + seekbar + audio/CC/tune/fullscreen) auto-hide after 3 s of playback (tap toggles them; kept visible while paused/buffering/dragging). **Center transport**: `replay_10` / big play-pause / `forward_10` float in a dark rounded pill in the middle of the screen, fading with the other controls. The bottom bar's background is a gradient mirroring the top bar (transparent → `black` 0.72), so both bars read at the same opacity. The player screen is **always immersive** (no system UI toggling during rotation — that fights the rotation animation and makes the video jitter); the bottom fullscreen button just forces landscape/portrait. Top-bar fullscreen button removed.
- **Audio track selection** (mute button replaced): the bottom bar's first button opens an "Audio tracks" bottom sheet listing every audio track from the native Media3 `currentTracks` (language · codec · channels · bitrate), with the active track check-marked. Picking a track calls `setAudioTrack` → native `TrackSelectionParameters` override → `onTracksChanged` re-emits → the top-bar audio chip (live codec + channel count) updates automatically. Native plumbing in `android/.../ExoPlayerView.kt` (`buildAudioTracks`, `selectAudioTrack`), pushed on every event as `audioTracks`/`selectedAudioTrack`; Dart model `ExoAudioTrack` in `lib/services/exo_player.dart`. Verified on-device: Sonic (DTS-HD MA + FLAC) switches DTS-HD → FLAC and the chip follows.
  - **Full track names**: the sheet prefers the container-provided track `label` (e.g. `DTS-HD MA 5.1`, `Commentary`) and appends the channel count unless the name already carries it; otherwise it composes `languageName(lang) · codec · channels`. `ExoAudioTrack` gained a `label` field; ISO-639 codes map to full English names via `languageName()` in `codec_info.dart`.
- **FLAC via FFmpeg + E-AC3 decoder workaround**: a custom `MediaCodecSelector` in `ExoPlayerView.kt` does two things: (1) returns no decoder for `audio/flac` so FLAC falls through to the bundled FFmpeg renderer — the platform MediaCodec FLAC decoder on some devices (incl. this OnePlus) allocates fixed 32 KiB input buffers and large FLAC frames (24-bit multichannel ~54 KiB) die with `DecoderInputBuffer$InsufficientCapacityException: Buffer too small`; (2) skips any `c2.dolby.eac3.decoder` for `audio/eac3`/`audio/eac3-joc` — on this OnePlus the codec2 resource manager repeatedly releases that hardware decoder as soon as it starts, so Media3's audio renderer spins in an endless re-init loop and **no AudioTrack is ever created (silent playback)**. With the Dolby component excluded, the AOSP software E-AC3 decoder is used and the renderer is stable. Verified on-device: Sonic FLAC plays continuously; an E-AC3 (Dolby Atmos, 5.1) track plays with an active AudioTrack (48 kHz, channelMask `0x3f`, no churn, no errors).
- **Subtitles — embedded + sideloaded with a full track picker**:
  - **Sibling auto-pairing** (`android/.../SubtitleFormats.kt` `findSiblingSubtitles`): on open, scans the video's folder and attaches **every** subtitle file as a Media3 `SubtitleConfiguration` (exact-filename-prefix match wins; ordered best-match first). The best match carries `SELECTION_FLAG_DEFAULT` so it's auto-selected; all others remain selectable in the picker. An explicitly passed `subtitleUri` still wins over pairing.
  - **`open()` path fix**: `lib/services/exo_player.dart` `open()` now sends `path` even when a `uri` is present — intent-opened files were dropping the path, so sibling pairing never fired. Verified on-device.
  - **Formats**: SRT, SSA/ASS, WebVTT, TTML/DFXP, SAMI (`.smi`), MicroDVD (`.sub`), MPL2 (`.mpl2`), SubViewer (auto-detected inside `.sub`). `SubtitleFormats` maps extension → MIME (incl. custom `application/x-sami`, `application/x-microdvd`, `application/x-mpl2`).
  - **Custom parsers** (`android/.../DreamSubtitleParserFactory.kt`): Media3's stock `DefaultSubtitleParserFactory` lacks SAMI/MicroDVD/MPL2/SubViewer, so `DreamSubtitleParserFactory` adds `SamiParser` and `FrameSubParser` (MicroDVD/MPL2/SubViewer modes) and delegates everything else (SubRip, SSA, WebVTT, TTML, PGS, VobSub, DVB, TX3G, CEA) to the default. Wired into both `DefaultMediaSourceFactory` and `DefaultExtractorsFactory` so the `SubtitleExtractor` picks it up.
  - **Charset handling**: Media3's text parsers decode UTF-8 only; `SubtitleFormats.toUtf8` detects BOM/strict-UTF-8 vs CP1252 and re-encodes non-UTF-8 sidecars to a cache file so legacy `.srt` files don't render as mojibake. `decodeToString` strips UTF-8 BOM for the custom parsers.
  - **Subtitle picker** (`lib/screens/player_screen.dart`): the bottom bar's CC button opens a sheet listing every subtitle track from native `currentTracks` (embedded container tracks + sideloaded files) plus Off. Labels append the format so sibling files read uniquely (`House.S02E04.eng · SRT`, `House.S02E04 · WebVTT`). Picking a track calls `selectSubtitleTrack` → native `TrackSelectionParameters` override; `selectedSubtitleTrack` re-emits → the CC button reflects the real selection.
  - **Note**: sibling auto-pairing needs `MANAGE_EXTERNAL_STORAGE` — without it, `listFiles()` only sees MediaStore-indexed files (SRT/TTML/SMI) and `.ass`/`.vtt`/`.sub`/`.mpl2` are silently skipped. Every `flutter install` re-revokes All Files Access on Android; re-grant via `adb shell am start -a android.settings.MANAGE_ALL_FILES_ACCESS_PERMISSION` (can't be granted via `adb shell pm grant` — this device blocks it).
  - Verified on-device (`House.S02E04` MKV + 7 sidecar formats): embedded PGS + all 7 sidecars attach, best-match `.eng.srt` auto-selected.
- **File browser (CX-Explorer style)** (`lib/screens/file_browser_screen.dart`): browse storage in-app and play any video without importing. **Back goes up one folder at a time** — only a folder whose path IS a root returns to the roots list; any other folder loads its parent (even when the parent is itself a root), so back from a folder inside a root lands on that root's contents, not on "Browse files". Folder icon in the home app bar opens it. Android side (`android/.../FileBrowser.kt`, channel `dreamplayer/files`): `hasAllFilesAccess` / `openAllFilesAccessSettings` / `getStorageRoots` (internal + SD card) / `listDirectory` (folders then video files, sorted, with sizes) / `pickFolder` (launches `ACTION_OPEN_DOCUMENT_TREE`, persistable URI grants stored in SharedPreferences as `dreamplayer.folderBookmarks`, result delivered via `MainActivity.onActivityResult` → `FileBrowser.onFolderPicked`) / `removeBookmark`. Bookmarked trees are appended to `getStorageRoots` with a `bookmarkId` and are listed through `DocumentFile` via synthetic paths `tree:<id>` / `tree:<id>/<relative>` (directory entries keep the synthetic path for back-navigation; video entries carry their `content://` document URI so `file_browser_screen.dart` passes it as `VideoItem.uri`, like the "Open with" flow). Requires **`MANAGE_EXTERNAL_STORAGE`** (All Files Access) on Android 11+ — the screen shows a "Grant access" button that opens the system settings and re-checks on app resume (the folder picker works without it, but browsing does not). iOS side (`ios/Runner/FileBrowser.swift`): sandboxed, so the root is the app's Documents folder plus **bookmarked folders picked via the system document picker** (`pickFolder` → `UIDocumentPickerViewController` for `.folder`, `removeBookmark`) — security-scoped bookmarks stored in UserDefaults keep picked folders (iCloud Drive, On My iPad, other providers) readable across launches, so videos outside the sandbox are browsed/played in-app without touching the Files app. The Dart screen shows a "Pick a folder" tile + per-bookmark remove at the root on **both** platforms (subtitle text is platform-specific). Tapping a video builds a `VideoItem` and pushes `PlayerScreen`. Verified on-device (Android): Internal storage → Download → video → Dolby Vision People plays with live HDR/codec chips.
- **"Open with" / file-explorer integration** (`AndroidManifest.xml` `ACTION_VIEW` intent-filters for `content`/`file` schemes + video MIME types incl. `video/*`, matroska, mpeg, ts, avi, wmv, octet-stream): tapping a video anywhere on the device now offers DreamPlayer. `MainActivity` resolves the intent (file path or `content://` URI + display name via `OpenableColumns`) and forwards it over the `dreamplayer/intent` channel (`getInitialIntent` on launch / `open` on `onNewIntent`); `lib/services/open_intent.dart` turns it into a `VideoItem` and pushes `PlayerScreen` via a global `appNavigatorKey` in `lib/app.dart`. `VideoItem` gained an optional `uri` (content URIs) with `path` now nullable; `ExoPlayerView` opens a raw URI when no path is available. Verified on-device: "Open with" chooser lists DreamPlayer and launches Dolby Vision playback.
  - **CX Explorer network-stream handoff**: CX hands SMB videos to players as `http://127.0.0.1:<port>/SMB/...` (its own local HTTP proxy), so the intent filter additionally declares `http`/`https`/empty schemes (`<data android:scheme=""/>`) + the full container-MIME matrix (`video/x-matroska`, `application/octet-stream`, `application/mpeg`, ... — a single filter, since per-vendor MIMEs differ), and `android:usesCleartextTraffic="true"`. Media3's `DefaultDataSource` handles file/content/asset itself and sends every other scheme (http/https) to the base factory, so `ExoPlayerView.kt` wires `DefaultHttpDataSource.Factory()` as that base — CX's proxy streams arrive there with no fallback and no extra code. Verified on-device via logcat: 4K HEVC lossless (3840×2176@60) streamed through CX's proxy decoded at a steady 60 fps / **0 discarded frames** for a full session (`c2.qti.hevc.decoder` telemetry), only jank = the app's cold start.
- **Home/settings status bar**: `RootShell` maps `MediaQuery.viewPadding.top` into `padding` (Android edge-to-edge reports `padding.top == 0`), so `SliverAppBar` never overlaps the status bar.
- **Library emptied of sample data**: the home library no longer shows hardcoded demo videos — it starts empty with a "Your library is empty" empty-state (file browser and "Open with" are the way in) until a real MediaStore scan lands. The dead "Scan for videos" button was removed.
- **Responsive grid** (`lib/screens/home_screen.dart`): column count and card height computed from screen width; card text is `Expanded`/`Flexible`. Text scaling clamped to 1.3x app-wide.
- **Native refresh rate** (`lib/services/display_refresh_rate.dart`): calls `FlutterDisplayMode.setHighRefreshRate()` on Android at startup.
- **Resume playback** (`lib/services/resume_store.dart`, shared_preferences): a video stopped mid-way resumes from where it left off on the next open. Position is bookmarked every ~5 s while playing, on pause, on app-background, and on player dispose; cleared when the video plays to the end. `ExoPlayerController.open` gained `startPositionMs` (native: iOS passes it as `startPosition` to `engine.load`, Android seeks before `play()`). Resume keys are the file path / content URI by default; sources whose playable URL rotates between sessions (iPad SMB token URLs) pass a stable `VideoItem.resumeKey` (`smb:<serverId>/<share>/<path>`). Skips trivial positions (<10 s) and "basically finished" ones (within 5 s of a known duration).
- **In-app SMB / LAN playback: REMOVED from both platforms (2026-08)**. The Android in-app SMB server browser (`smb_screen.dart`, `smb_client.dart`, `SMBClient.kt`, `SmbDataSource.kt`, channel `dreamplayer/smb`, jcifs-ng) was deleted from the app — on Android the user's day-to-day workflow plays NAS files via **CX Explorer's network share → "Open with" → DreamPlayer**, which streams over CX's local HTTP proxy at full speed (see the CX handoff note above). The **Network shares** home-screen entry is **hidden on ALL platforms now** — the iPad's AMSMB2-based in-app browser (`SmbScreen`) kept crashing on audio-track switch (see the roadmap section below), and the file-browser + Files-app "Open with" + bookmarked-folder paths cover both local and NAS workflows without the crash. The `lib/screens/smb_screen.dart` + `lib/services/smb_client.dart` code and the iOS `SMBClient.swift` runtime are still in the tree (unreachable from the UI) — the knowledge is preserved in the roadmap section below for future development. **Lesson learned on-device**: jcifs-ng's streaming read size is bound by three interlocking properties (`snd_buf_size`/`rcv_buf_size`/`transaction_buf_size`, defaults 65535); raising only the first two did nothing (still ~64 KB reads / ~5 MB/s with constant ring-buffer stalls), and raising `transaction_buf_size` to 8 MiB made the NAS reject reads with `STATUS_INVALID_PARAMETER` ("The parameter is incorrect"); 1 MiB was still rejected. Do not raise buffers past what the NAS's negotiated `MaxReadSize` accepts.
- Tests: 43 (`flutter test`) incl. no-overflow checks on small phone, tablet, landscape, and 2.0x text scale, a file-browser back-navigation test (back goes up one folder at a time; a folder inside a root lands on that root's contents, not the roots list), and an About → "Open-source licenses" navigation test.
- **Licensing**: the app is **GPLv3** because the Android build links `nextlib-media3ext` (GPLv3 FFmpeg extension). `LICENSE` (GPLv3 text) + `NOTICE` (third-party components: Media3 Apache-2.0, nextlib GPL-3.0, AetherEngine LGPL-3.0 + Apple Store exception, FFmpegBuild LGPL-2.1+, AMSMB2 MIT, Flutter BSD-3-Clause, pub plugins MIT/BSD). The About section of Settings opens `licenses_screen.dart`, which lists every component and its license.
- **Donations**: Settings → **Support** lists two donation channels (Razorpay, GitHub Sponsors) via `lib/services/support_links.dart` (`url_launcher`). **Razorpay is set** (`https://rzp.io/rzp/cZ5afqVG`, a live payment link → `plink_TOrUqMDPRxYQFp`) and **GitHub Sponsors is set** (`https://github.com/sponsors/mangeshghodke/`). README has matching badges + a Support section.

## Roadmap

### SMB / network shares (Android + iPad)

Play files from LAN/NAS SMB shares in-app, mirroring the existing file-browser pattern.

**Status: REMOVED from both platforms (2026-08).** An Android in-app SMB v1 (browse
+ stream) was implemented and verified against the real NAS, then **removed from the app** —
on Android the user's workflow is CX Explorer's network share → "Open with" → DreamPlayer
(streams over CX's local HTTP proxy at full speed; no in-app SMB needed on Android). **The iPad
in-app SMB browser (AMSMB2 + AetherEngineSMB) shipped and was verified on-device, but its home
entry is now HIDDEN (2026-08-13): picking a different audio track on an SMB stream could crash
the app** (see "iOS/iPad status" below). The file-browser + Files-app "Open with" + bookmarked
folders cover local and NAS workflows, so SMB playback was unreachable temporarily. The complete
SMB knowledge below is the blueprint if in-app SMB ever returns.

**Architecture**
- New native module per platform exposing a MethodChannel (same shape as `FileBrowser.kt` / `dreamplayer/files`):
  - Android: `SMBClient.kt` — channel `dreamplayer/smb`
  - iOS/iPad: `SMBClient.swift` — same channel
- Dart: `lib/services/smb_client.dart` (models + channel wrapper) + `lib/screens/smb_screen.dart` (server list → shares → folders → tap video → `PlayerScreen`).
- Playback passes an `smb://` URI through the existing `uri` path in `VideoItem` (like the "Open with" flow).

**Libraries**
| Platform | Choice | Why |
|---|---|---|
| Android | **jcifs-ng** (SMB2/3 only) | Nova's and CX File Explorer's SMB library; measured ~75 MB/s vs ~4–6 MB/s for smbj on the NAS. |
| Android (optional) | jcifs-ng SMB1 | SMB1 legacy devices only (disabled by default; SMB1 support is behind a config flag) |
| iPad | **AMSMB2** / **SwiftSMB** (Swift wrapper over **libsmb2**, C) | Only mature SMB2/3 path on iOS; libsmb2 is a serious candidate (Nova has discussed it) |
- **Licensing**: libsmb2 is LGPL-2.1 (constrains App Store distribution — needs relinkable/replaceable lib); app already ships GPLv3 FFmpeg extension so not a new concern for Android.

**Features**
1. *Servers*: add/edit/delete saved servers (name, host/IP, port 445, user, password, or Guest); credentials in Keychain (iOS) / Android Keystore (EncryptedSharedPreferences), never plaintext; LAN auto-discovery (broadcast/workgroup) + manual IP fallback; test connection + quick connect; saved-server status dot (online/offline).
2. *Browsing* (CX-Explorer style): server → shares → folders → files; breadcrumbs + up-nav; folders first, sorted by name/size/date; show size + modified date; player back returns to same folder.
3. *Playback*: direct streaming (no download) — Android = custom ExoPlayer `DataSource` over jcifs-ng seekable reads; iPad = `AVAssetResourceLoaderDelegate` serving bytes from the SMB stream; full seek; existing live HDR/codec chips unchanged; play-next-episode in folder; optional prefetch/cache-ahead setting + reconnect-on-drop/resume for high-bitrate files.
4. *Extras*: auto-pair subtitles from same folder (`.srt`/`.ass`); pin recently-used servers on home screen; DNS/WINS hostname resolution for NAS names.
- **Scope (v1)**: manual server add + Guest/basic auth + browse + stream + play-next. Add discovery + subtitles after.
- **Status**: v1 core landed (Android): discovery, status dots, play-next-episode and subtitle auto-pair are implemented and the app is running on-device; the Nova-style read-ahead ring buffer is implemented but **not yet verified against a real NAS**. Remaining: **full NAS verify of streaming/seek + subtitles + play-next** (share browsing is verified), Reconnect-on-drop/resume for high bitrates, and the iPad path (needs an SMB2 client on the Swift side).
- **iOS/iPad status (2026-08-13 — home entry HIDDEN)**: the in-app SMB browser + playback landed for iPad via **AMSMB2** (`ios/Runner/SMBClient.swift`, channel `dreamplayer/smb`, same Dart `SmbClient` contract as the removed Android one) + **AetherEngineSMB** (the engine's official SMB product — `SMBConnection` + `SMBIOReader`). `openShare` returns a per-file `dreamplayersmb://<token>.<ext>` URL; the platform view resolves the token to the live `SMBConnection` and loads it as a custom `IOReader` source (`engine.load(source: .custom(SMBIOReader(...), formatHint: nil))` — the demuxer probes the container itself). `closeShare(serverId)` closes every connection for the server. Servers persist in UserDefaults (passwords in Keychain, never to Dart); shares list via `listShares` + manual add-share; directory listing sorts folders-then-videos and auto-pairs sibling subtitles (`subtitlePath` downloaded to a temp file — subtitles are small — and returned as a `file://` URL for `ExternalSubtitleTrack`). LAN scan (`discoverServers`) probes the local /24 on port 445. Registered in `AppDelegate`; `NSBonjourServices` + `NSLocalNetworkUsageDescription` in Info.plist; AMSMB2 (SPM 4.0.0) + AetherEngineSMB (product of the AetherEngine package, added to the Runner **Embed Frameworks** phase) in the project. **Why not the loopback HTTP proxy:** AetherEngine's bundled FFmpeg has **no network stack** — it plays remote URLs through its own "loopback producer" with one long-lived connection + open-ended ranges; a hand-rolled HTTP server (`Connection: close`, no keep-alive) mismatched that protocol and "Share connects but video won't open" persisted across ATS / extension+Content-Type / connect-race fixes (v0.0.3). AetherEngineSMB is the engine-native path for NAS/SMB sources. **Why the home entry is hidden (2026-08-13):** picking a different audio track on an SMB stream could **crash the app** on-device. The EPERM failure was fixed (`reopenSMBStream` + `SMBClient.reconnect` mint a fresh connection)/the buffering spinner got `BufferedSMBReader`, but a hard crash remained in the reopen/teardown path (stale-connection close racing an in-flight read). Since local playback is smooth and NAS files reach the app via CX/Files "Open with", the SMB home entry was removed on all platforms with no feature-loss workaround; the code stays in the tree (`smb_screen.dart`, `smb_client.dart`, `SMBClient.swift`, `AvPlayerView` SMB paths) as a future rebuild blueprint. To revive: fix the teardown race, requiring the iPad crash report/console at the moment of the audio-track tap.
  - **Gotcha fixed on-device — dynamic SPM framework not embedded**: AMSMB2's package product is `type: .dynamic`, so linking it into Runner is NOT enough. It must ALSO be added to the Runner target's **Embed Frameworks** copy phase (`PBXCopyFilesBuildPhase`, `dstSubfolderSpec = 10`) as a `PBXBuildFile` with `productRef` + `settings = {ATTRIBUTES = (CodeSignOnCopy, RemoveHeadersOnCopy); }`. Without that, `AMSMB2.framework` is missing from `Runner.app/Frameworks/` (the binary still has an `@rpath/AMSMB2.framework/AMSMB2` load command, so the build passes but dyld crashes at launch with "image not found"). Transitive dynamic products (e.g. FFmpegBuild's xcframeworks pulled in by AetherEngine) are auto-embedded; direct package products added by hand to the Frameworks phase are not. **AetherEngineSMB is the opposite — a STATIC library product (its `Package.swift` `products` entry has no `type:`, so the default static applies; same for its `SMBClient` dependency). It must be in the Frameworks (link) phase + `packageProductDependencies`, and must NOT be added to the Embed Frameworks copy phase — with no `.framework` file to embed, xcodebuild fails with `The file "AetherEngineSMB" couldn't be opened because there is no such file`.**
  - **"Share connects but video won't open" fix (2026-08-12, superseded)**: the v0.0.3 loopback-HTTP fixes — (1) ATS (`NSAllowsLocalNetworking` for the `http://127.0.0.1` stream URL, since the native AVPlayer path honors ATS), (2) extension + `Content-Type` on the token URL, (3) synchronous connect before returning the URL — did NOT fix playback on-device; the loopback server was retired in v0.0.4 in favor of AetherEngineSMB (see above). The ATS entry stays in Info.plist (harmless).

## CI / Deployment

- **iOS builds happen in GitHub Actions** (user has no Mac). Workflow: `.github/workflows/ios.yml`
  - Runs on `macos-latest`, builds unsigned IPA artifact always.
  - Signed build + TestFlight upload run only when secrets are configured.
  - Secrets needed: `IOS_CERT_BASE64`, `IOS_CERT_PASSWORD`, `IOS_PROFILE_BASE64`, `APPSTORE_API_KEY`, `APPSTORE_API_KEY_ID`, `APPSTORE_ISSUER_ID`.
- **GitHub Releases** (`.github/workflows/release.yml`): push a `v*` tag (`git tag v0.0.1 && git push origin v0.0.1`) → builds the **universal** release APK + **split-per-abi** APKs (`arm64-v8a`, `armeabi-v7a`, `x86_64`) + AAB on `ubuntu-latest` and the unsigned iOS IPA (`DreamPlayer.ipa` — name deliberately omits "unsigned") on `macos-latest`, then creates the GitHub Release on the tag with `.github/release_notes.md` (Android architecture guide + iOS sideload guide) plus auto-generated change notes, and attaches all artifacts. Android artifacts are renamed `DreamPlayer-<version>-universal.apk`, `DreamPlayer-<version>-<arch>.apk`, `DreamPlayer-<version>.aab`. App version starts at **0.0.1** (`pubspec.yaml` `version: 0.0.1+1` → Android versionName/versionCode + iOS CFBundleShortVersionString/CFBundleVersion) and must be bumped per release to match the tag. Android APKs are currently **debug-signed** (`build.gradle.kts` uses the debug signing config for release builds) — fine for sideloading, but add a real signing config before Play Store / wide distribution.
- **Bundle ID (iOS)**: `com.dreamplayer.app`. **App display name**: `DreamPlayer`.
- **Android**: app label `DreamPlayer`; package `com.dreamplayer.app` (matches iOS bundle ID `com.dreamplayer.app`). Build/test locally on the phone.

## Repository / Git

- Remote: `https://github.com/mangeshghodke/DreamPlayer.git`
- Branch: `main`
- Never commit secrets.

## Commands

```bash
flutter pub get          # fetch dependencies
flutter analyze          # static analysis
flutter test             # run tests
flutter run              # run on connected Android phone (USB debugging)
flutter run --release    # test real-world smoothness (debug is jankier)
flutter build apk --debug --target-platform android-arm64 && flutter install --debug -d <device-id>
flutter build apk        # release APK (use --split-per-abi)
flutter build appbundle  # for Play Store
adb shell monkey -p com.dreamplayer.app -c android.intent.category.LAUNCHER 1   # launch app
adb shell dumpsys SurfaceFlinger | grep -a activeMode                                  # check refresh rate
```

## Display & smoothness (native refresh rate)

- **Android**: `flutter_displaymode` selects the display's highest refresh rate at app startup (`lib/services/display_refresh_rate.dart`). Many Android devices default apps to 60 Hz even on 90/120/144 Hz panels. Verified: panel runs 120 Hz during animations, 60 Hz when idle.
- **iOS/iPad Pro**: ProMotion 120 Hz is unlocked via `CADisableMinimumFrameDurationOnPhone = true` in `ios/Runner/Info.plist` (already set).
- **Playback cadence**: ExoPlayer renders at the video's FPS onto the platform-view SurfaceView. Revisit frame pacing once smoothness is assessed on-device.

## Project layout

```
lib/
  main.dart                     # entry point (native refresh rate, runs app)
  app.dart                      # root MaterialApp, dark theme, text-scale clamp, nav shell
  theme/app_theme.dart          # colors, dark theme (video apps are dark)
  models/
    video_item.dart             # VideoItem + codec label getters
    hdr_format.dart             # HdrFormat enum (SDR/HDR10/HDR10+/DV/HLG)
  utils/codec_info.dart         # HDR detection + codec -> label mapping + live label merge
  services/display_refresh_rate.dart  # high refresh rate selection (Android)
  services/exo_player.dart        # ExoPlayerController + ExoPlayerView platform view
  services/smb_client.dart        # SMB channel wrapper (iPad in-app shares; same contract as removed Android one)
  screens/
    home_screen.dart            # library grid (adaptive columns, empty state until scanning lands)
    player_screen.dart          # ExoPlayer/Media3 playback + live codec/HDR chips + controls + subtitle/audio pickers
    smb_screen.dart             # in-app SMB share browser (server list → shares → folders → play)
    settings_screen.dart        # settings list
  widgets/
    video_card.dart             # library card with HDR/audio badges
    format_chip.dart            # colored codec/HDR chip
android/app/src/main/kotlin/com/dreamplayer/app/
  ExoPlayerView.kt              # native PlayerView platform view + channels (open/play/seek/tracks/subtitles)
  SubtitleFormats.kt            # extension->MIME map, sibling auto-pairing, charset detection, UTF-8 re-encode
  DreamSubtitleParserFactory.kt # SAMI/MicroDVD/MPL2/SubViewer parsers + default delegate
  FileBrowser.kt                # device storage browsing channel
  MainActivity.kt               # registers platform views + "Open with" intent handling
ios/Runner/
  AvPlayerView.swift            # AetherEngine platform view + channels (same contract as ExoPlayerView.kt); host SubtitleOverlayView; SMB streams load via BufferedSMBReader custom source
  BufferedSMBReader.swift       # read-ahead sliding-window IOReader (32 MiB) for SMB playback
  SMBClient.swift               # AMSMB2 SMB client (channel dreamplayer/smb); AetherEngineSMB playback connections; keychain passwords + LAN discovery
  FileBrowser.swift             # Documents-folder browsing channel (same contract as FileBrowser.kt)
  IntentBridge.swift            # "Open with" intent channel (same contract as MainActivity.kt)
  AppDelegate.swift             # registers the AvPlayerView factory + files/intent/smb channels
  SceneDelegate.swift           # forwards scene-opened URLs to IntentBridge
test/
  widget_test.dart              # shell/navigation/overflow tests
  codec_info_test.dart          # HDR + codec formatting unit tests
```

## Workflow for the user (no Mac)

1. Develop + test on Android phone (USB debugging, `flutter run`).
2. Commit/push to `main`; iOS workflow in GitHub Actions builds the iPad version.
3. Later: configure code-signing secrets + TestFlight for installing on iPad Pro M2.

---
> Source: [mangeshghodke/DreamPlayer](https://github.com/mangeshghodke/DreamPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

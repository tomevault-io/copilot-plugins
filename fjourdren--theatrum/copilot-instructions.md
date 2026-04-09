## theatrum

> Theatrum is a streaming server built in Go supporting:

# Theatrum - Claude Code Context

## Project Overview

Theatrum is a streaming server built in Go supporting:
- Video on Demand (VOD) with adaptive bitrate streaming
- Multiple quality profiles (low, medium, high)
- HLS (HTTP Live Streaming) protocol
- MPEG-DASH protocol (standalone or dual-mode with HLS)
- Live RTMP streaming
- Restreaming from external URLs (RTMP or any FFmpeg-compatible protocol)

## Architecture

**Pattern:** Hexagonal (Ports & Adapters) Architecture

```
src/
├── cmd/main.go              # Entry point with DI container (uber/dig)
├── constants/               # App constants, paths, video settings
├── shared/
│   └── ffmpegargs/          # Shared FFmpeg argument builders (filter, codecs, stream map)
├── domain/
│   ├── models/              # Stream, Quality, Application, Server, Distribution
│   ├── services/            # ApplicationService, StreamService, EncodeService, PathTemplateService, ViewerTracker
│   ├── repositories/        # Port interfaces (ConfigurationPort, EncoderPort, StoragePort)
│   └── jobs/                # EncodeJobQueue, VideoUnencodedDetector
└── adapters/
    ├── driven/              # Output adapters
    │   ├── ffmpegEncoder/   # FFmpeg encoding implementation
    │   │   └── repositories/# EncoderPort implementation (VOD encoding)
    │   ├── fileAccess/      # File system operations
    │   ├── metrics/         # Prometheus metrics collector
    │   └── yamlConfigFile/  # YAML configuration loader
    └── driver/              # Input adapters
        ├── http/            # HTTP/HLS/DASH server
        ├── restream/        # Restream manager (pull from external URLs)
        ├── rtmp/            # RTMP streaming server
        └── ports/           # Port interfaces for drivers
```

## RTMP Implementation

### Components

| Component | Path | Description |
|-----------|------|-------------|
| Server | `adapters/driver/rtmp/rtmpServer.go` | Main RTMP server lifecycle |
| Handler | `adapters/driver/rtmp/handlers/handler.go` | Connection handling, publish flow |
| Manager | `adapters/driver/rtmp/management/manager.go` | Active stream management |
| Process | `adapters/driver/rtmp/management/process.go` | FFmpeg process wrapper |
| Auth | `adapters/driver/rtmp/auth/` | URL pattern matching, XOR auth |
| FLV | `adapters/driver/rtmp/flv/` | FLV tag serialization |
| Thumbnail | `adapters/driver/rtmp/management/thumbnail.go` | Periodic PNG thumbnail extraction |

### Data Flow

```
RTMP Client (OBS, etc.)
    ↓
RTMP Server (port 1935)
    ↓
Handler.OnConnect() - Validates TCURL against channel patterns
    ↓
Handler.OnPublish() - XOR token authentication, resolves record path
    ↓
StreamManager.GetOrCreateStream() - Creates/reuses FFmpeg process
    ↓
FLV Writer - Serializes RTMP frames to FLV format
    ↓
FFmpeg (stdin pipe) - Converts FLV to HLS, DASH, or both (passthrough or multi-quality)
    ↓
Output: HLS (.m3u8 + .ts), DASH (.mpd + .m4s), or Dual (both via CMAF .m4s)
    ↓
HTTP Server (port 8080) - Serves HLS/DASH to viewers
    ↓
On stream end: cleanup (delete files) or recording (generate VOD playlist + move to record path)
```

### Authentication

Live streams use configurable XOR-based authentication via `auth_token_template`:

1. Client connects to `rtmp://server/user/{username}`
2. TCURL matched against channel patterns in config
3. Server builds XOR input from `auth_token_template` by replacing `{var}` placeholders with URL values
4. On publish, client sends `publishingName = XOR(auth_input, live_stream_key)` (hex encoded)
5. Server validates by computing expected token

**Required fields for live streams:**
- `live_stream_key` - Secret key for XOR operation
- `auth_token_template` - Template specifying which URL variables to use (e.g., `{username}`, `{room_id}{username}`)

## Restream Implementation

### Overview

Restreams pull from an external URL (RTMP or any FFmpeg-compatible protocol) and output HLS/DASH segments. Unlike live streams (push model via RTMP), restreams use a pull model where FFmpeg directly reads from the source URL.

```
External URL → FFmpeg -i <url> → HLS/DASH segments → HTTP server serves to viewers
```

### Components

| Component | Path | Description |
|-----------|------|-------------|
| RestreamManager | `adapters/driver/restream/restreamManager.go` | Manages all restream channels |
| RestreamPort | `adapters/driver/ports/restream.go` | Port interface |

### Behavior

- **Auto-start**: All restream channels start on server boot
- **Auto-reconnect**: Exponential backoff (1s → 2s → 4s → ... → 30s max) on source failure; resets after 30s of successful streaming
- **No user variables**: Channel paths must be literal (no `{var}` placeholders), only `{%FUNC%}` builtins allowed
- **Recording/viewers/views**: Same support as live streams
- **FFmpeg command**: Reuses the same command builders as live streams, but with `-i <url>` instead of `-f flv -i pipe:0`

### Config Constraints

- `source_url` required, non-empty
- Must NOT have `live_stream_key`, `auth_token_template`, `video_input_path`, `delete_after_encoding`
- Channel key, `path`, and `record.path` must not contain `{var}` placeholders (only `{%FUNC%}` builtins)

## Configuration

Copy `config.yml.example` to `config.yml`:

```yaml
server:
  http: 8080
  rtmp: 1935

channels:
  # Passthrough (codec copy, no transcoding, lowest latency)
  "/user/{username}":
    stream:
      type: live
      path: "livestreams/{username}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"  # REQUIRED for live streams
      distribution:
        hls:
          segment_duration: 2
          window_size: 3     # Segments in live playlist (default: 3)

  # Multi-quality transcoding (adaptive bitrate, requires CPU)
  "/premium/{username}":
    stream:
      type: live
      path: "livestreams/{username}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      qualities:           # Optional: omit for passthrough
        low: ...
        medium: ...
        high: ...
      distribution:
        hls:
          segment_duration: 2
          window_size: 5

  # Live stream with recording (files moved to record.path)
  "/recorded/{username}":
    stream:
      type: live
      path: "live/{username}/{%STARTING_DATE%}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      distribution:
        hls:
          segment_duration: 2
          window_size: 5
      record:
        enabled: true
        path: "recordings/{username}/{%STARTING_DATE%}"

  # Live stream with in-place recording (files stay in stream.path)
  "/inplace/{username}":
    stream:
      type: live
      path: "live/{username}/{%STARTING_DATE%}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      distribution:
        hls:
          segment_duration: 2
          window_size: 5
      record:
        enabled: true

  # Live stream with viewer/view tracking
  "/tracked/{username}":
    stream:
      type: live
      path: "live/{username}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      distribution:
        hls:
          segment_duration: 2
          window_size: 5
      viewers:              # Concurrent viewer count (live streams only)
        enabled: true
        window: 30          # Minimum seconds of watching before counted (default: 30)
      views:                # Total view count (all stream types)
        enabled: true
        window: 30          # Minimum seconds of watching before counted (default: 30)
      thumbnail:            # Periodic PNG thumbnail (live streams only)
        enabled: true
        interval: 5         # Generate a thumbnail every 5 seconds

  # DASH-only live stream
  "/dash/{username}":
    stream:
      type: live
      path: "live/{username}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      distribution:
        dash:
          segment_duration: 2
          window_size: 5

  # Dual-mode live stream (HLS + DASH from a single FFmpeg process)
  "/dual/{username}":
    stream:
      type: live
      path: "live/{username}"
      live_stream_key: "your-secret-key"
      auth_token_template: "{username}"
      qualities:
        low: ...
        medium: ...
        high: ...
      distribution:
        hls:
          segment_duration: 2
          window_size: 5
        dash:
          segment_duration: 2  # Must match HLS segment_duration in dual mode
          window_size: 5

  # Restream: pull from external URL (passthrough)
  "/restream/mystream":
    stream:
      type: restream
      source_url: "rtmp://external-server/live/stream_key"
      path: "restream/mystream"
      distribution:
        hls:
          segment_duration: 2
          window_size: 5

  # Restream with recording
  "/restream/recorded":
    stream:
      type: restream
      source_url: "rtmp://external-server/live/stream_key"
      path: "restream/recorded/{%STARTING_DATE%}"
      distribution:
        hls:
          segment_duration: 2
          window_size: 5
      record:
        enabled: true
        path: "recordings/restream/{%STARTING_DATE%}"
```

### Distribution Modes

The `distribution` block supports three modes based on which formats are configured:

| Mode | Config | Muxer | Segments | Output files |
|------|--------|-------|----------|--------------|
| **HLS-only** | `hls` only | `-f hls` | `.ts` | `master.m3u8` + per-quality `playlist.m3u8` |
| **DASH-only** | `dash` only | `-f dash` | `.m4s` | `manifest.mpd` + init/chunk `.m4s` |
| **Dual** | both `hls` + `dash` | `-f dash -hls_playlist 1` | `.m4s` (fMP4/CMAF) | `manifest.mpd` + `master.m3u8` + `.m4s` |

**HLS-only modes:**
- **Without `qualities`** (passthrough): codec copy, single playlist at `{path}/default/playlist.m3u8`
- **With `qualities`** (transcoding): multi-quality HLS with `master.m3u8` + per-quality subdirs (`low/`, `medium/`, `high/`), uses `-preset veryfast -tune zerolatency` for real-time encoding

**DASH / Dual modes:**
- Flat directory layout (no quality subdirs) — FFmpeg manages representations internally
- In dual mode, `segment_duration` must match between `hls` and `dash` (enforced by config validation)
- Dual mode shares fMP4/CMAF `.m4s` segments between both manifests
- Recording: FFmpeg finalizes manifests on clean exit; no additional VOD playlist generation needed

**Implementation:** `OutputMode` enum in `src/adapters/driver/rtmp/management/process.go` (`OutputModeHLS`, `OutputModeDASH`, `OutputModeDual`) determines which FFmpeg command is built. `DetermineOutputMode()` derives the mode from the `Distribution` config.

### Recording

Live streams can optionally be recorded. When `record.enabled` is `true`:
- **During stream**: All segments are kept on disk (no deletion), but only the last `window_size` segments appear in the live playlist
- **On stream end**: A VOD playlist is generated from all segments

**Recording modes:**

| `record.enabled` | `record.path` | After stream ends |
|---|---|---|
| `false` (default) | N/A | Files deleted after `cleanup_delay` |
| `true` | set | VOD playlist generated, files moved to `record.path` |
| `true` | omitted | VOD playlist generated in-place, files stay in `stream.path` |

When recording is disabled (default):
- **During stream**: Sliding window with only the last `window_size` segments on disk
- **On stream end**: All remaining files are deleted after `cleanup_delay`

`record.path` is optional. When provided, it supports the same `{var}` and `{%FUNC%}` placeholders as `stream.path`. Built-in functions resolve to the same values as the stream's path within the same session. When omitted, files remain in `stream.path` after the stream ends (in-place recording).

**Recording with DASH/Dual modes:** FFmpeg finalizes the MPD manifest (and HLS playlists in dual mode) on clean exit. No additional VOD playlist generation is performed — files are simply moved (or kept in-place) as-is.

### Path Template System

Path templates support two types of placeholders:

- **User variables** `{var}` - Extracted from URL patterns (e.g., `{username}`, `{room_id}`)
- **Built-in functions** `{%FUNC%}` - Auto-generated values computed at resolution time

**Available built-in functions:**

| Function | Description | Example output |
|----------|-------------|----------------|
| `{%STARTING_DATE%}` | Current date/time (format: `2006-01-02_15-04-05`) | `2026-02-07_15-30-00` |
| `{%UUID%}` | Random UUID v4 | `550e8400-e29b-41d4-a716-446655440000` |

**Example usage in config:**
```yaml
path: "livestreams/{username}/{%STARTING_DATE%}"    # → livestreams/alice/2026-02-07_15-30-00
path: "recordings/{%UUID%}"                          # → recordings/550e8400-e29b-41d4-a716-446655440000
```

**Implementation details:**
- Built-in functions are resolved first (phase 1), then user variables (phase 2)
- Both phases sanitize values through `sanitizeValue()` (alphanumeric, `_`, `-`, `.` only)
- Registry lives in `PathTemplateService.builtinFuncs`; new functions can be added via `RegisterBuiltinFunc()`
- Constants defined in `src/constants/templateConstantes.go`

### Viewer & View Tracking

Theatrum tracks concurrent viewers and total views per stream by monitoring `.ts` and `.m4s` segment requests from unique client IPs. Both viewers and views use a **delayed window** — they only count after a client has been watching continuously for `window` seconds.

**Components:**
- `ViewerTracker` service (`src/domain/services/viewerTracker.go`) — tracks per-stream viewer/view data with delayed counting, persists view counts to disk
- `StreamHandler` (`src/adapters/driver/http/handlers/streamHandler.go`) — calls tracker on `.ts`/`.m4s` requests, serves `viewers.txt`/`views.txt`
- Cleanup on stream end via `StreamProcess.Stop()` calling `ViewerTracker.UnregisterStream()`

**Endpoints** (served alongside `master.m3u8` or `manifest.mpd`):
- `viewers.txt` — concurrent viewer count (live streams only), returns 404 if disabled
- `views.txt` — total view count (all stream types), returns 404 if disabled

**Delayed window behavior:**
- Each client IP starts a new session on first `.ts`/`.m4s` request (or after inactivity >= `window`)
- **Viewers**: a client only appears in the viewer count after watching continuously for `window` seconds. If they stop requesting segments for `window` seconds, their session resets
- **Views**: a view is only counted once a client has watched continuously for `window` seconds. Each session can increment the view count at most once
- `window: 0` counts immediately on first request (same as instant behavior)

**Tracking key:** Fully resolved stream path (e.g., `live/alice/2026-02-14_12-30-45`)

**Client identification:** `X-Forwarded-For` header (first IP) or `RemoteAddr` (port stripped)

**View count persistence:**
- View counts are persisted to disk at `{VideoDir}/{trackingKey}/views.txt`
- Counts are flushed to disk periodically (every 10s when dirty) and on stream end (`UnregisterStream()`)
- On stream start, `ViewerTracker` seeds `totalViews` from the existing file on disk
- When a stream is not in memory, `GetViewCount()` falls back to reading from disk
- Viewer counts (concurrent) are ephemeral and correctly reset — only views are persisted
- During non-recording stream cleanup, all files are deleted (including `views.txt`)
- During recording with `record.path`, `views.txt` is moved alongside other files to the recording directory
- During in-place recording, `views.txt` stays in `stream.path` with other files

### Thumbnail Generation

Theatrum can periodically generate a PNG thumbnail from a live stream, enabling clients (web players, stream directories) to show a live preview.

**Config fields:**
```yaml
thumbnail:
  enabled: true
  interval: 5    # seconds between captures
```

**Restriction:** Live streams only (`type: live`). Config validation rejects `thumbnail.enabled` on other stream types.

**Implementation:** A separate FFmpeg process (`ThumbnailGenerator` in `src/adapters/driver/rtmp/management/thumbnail.go`) runs alongside the main streaming FFmpeg. Every `interval` seconds it:
1. Finds the latest segment file on disk (`.ts` for HLS, `.m4s` for DASH/Dual)
2. For multi-quality HLS, selects the highest available quality dir (high > medium > low)
3. For `.m4s` segments, concatenates the init segment (`init-stream0.m4s`) with the chunk via FFmpeg's `concat:` protocol
4. Runs `ffmpeg -frames:v 1 -update 1` to extract one frame as PNG
5. Writes atomically via `.tmp` + rename to `{streamRootDir}/thumbnail.png`

**HTTP endpoint:** `thumbnail.png` is served alongside `master.m3u8`/`manifest.mpd` at the channel path. Returns 404 when `thumbnail.enabled` is `false`. Content-Type: `image/png`, Cache-Control: `public, max-age=2`.

**Lifecycle:**
- Started by `Manager.createNewStream()` after the stream becomes active
- Stopped by `StreamProcess.Stop()` before shutting down the main FFmpeg process

**Recording/cleanup behavior:**
- **No recording**: `os.RemoveAll(streamRootDir)` deletes `thumbnail.png` with all other files
- **HLS passthrough recording** (with `record.path`): `thumbnail.png` is explicitly moved from `streamRootDir` to `recordDir` alongside `views.txt`
- **HLS multi-quality / DASH / Dual recording**: `moveContents(streamRootDir, recordDir)` moves `thumbnail.png` with everything else
- **In-place recording**: `thumbnail.png` stays in `streamRootDir` with other files

## Stream Types

- `video_encoded` - Pre-encoded VOD content
- `video_unencoded` - Raw videos to be encoded
- `live` - Live RTMP streams (passthrough or multi-quality transcoding)
- `restream` - Pull from external URLs and re-broadcast (passthrough or multi-quality)

## Development

```bash
# Build
go build -o theatrum ./src/cmd/main.go

# Run
./theatrum

# Test RTMP (using FFmpeg)
ffmpeg -re -i input.mp4 -c copy -f flv "rtmp://localhost/user/myuser"
```

## Metrics

Prometheus metrics are exposed at `GET /metrics` on the HTTP port. The `Metrics` struct in `src/adapters/driven/metrics/metrics.go` holds all Prometheus collectors and is created once by `NewMetrics()`, then injected via `dig` into all components that need instrumentation (HTTP server, RTMP handler, stream processes, encode job queue). All custom metrics are prefixed with `theatrum_`. The `ResponseWriter` wrapper in the same package captures HTTP status codes and bytes written for instrumentation.

Stream-lifecycle metrics (`theatrum_stream_duration_seconds`, `theatrum_ffmpeg_exits_total`, `theatrum_recordings_total`, `theatrum_rtmp_received_bytes_total`, `theatrum_rtmp_received_frames_total`) carry a `stream_path` label with the fully resolved tracking key (e.g., `live/alice/2026-02-14_12-30-45`) for per-stream monitoring.

A custom `ViewerCollector` (`src/adapters/driven/metrics/viewerCollector.go`) implements `prometheus.Collector` to export per-stream viewer and view counts. It queries `ViewerTracker.GetAllStreamStats()` on each Prometheus scrape, emitting `theatrum_stream_viewers` and `theatrum_stream_views` gauges with a `stream_path` label. Using const metrics means stale streams disappear automatically when they are unregistered.

## Key Dependencies

- `github.com/prometheus/client_golang` - Prometheus metrics
- `github.com/yutopp/go-rtmp` - RTMP protocol implementation
- `go.uber.org/dig` - Dependency injection
- `gopkg.in/yaml.v3` - YAML configuration parsing

## Conventions

- **Files**: camelCase for Go files (e.g., `streamService.go`)
- **Packages**: lowercase single words (e.g., `handlers`, `models`)
- **Interfaces**: Port interfaces suffixed with 'Port' (e.g., `ConfigurationPort`, `EncoderPort`)
- **Tests**: Same package with `_test.go` suffix

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fjourdren)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fjourdren)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## deer-stop

> Wildlife camera pipeline: RTSP stream → YOLO inference → MJPEG stream + clip recorder.

# DeerStop - Claude Memory

## Project Overview

Wildlife camera pipeline: RTSP stream → YOLO inference → MJPEG stream + clip recorder.
Three components: detection server (Python, edge device "astrapi"), node server (WebRTC signaling/TURN), iOS app (SwiftUI).
The iOS app is a native remote client — inference stays on the edge device; the app is purely for viewing/browsing.

## Architecture

- **Three-tier**: Edge device (detection + inference) → Relay server (Node) → iOS app
- **Edge device**: Raspberry Pi / embedded device with hostname `astrapi`, running systemd service `deerstop`
- **RTSP source**: `rtsp://admin:...@192.168.1.241:554/h264Preview_01_sub` — has both H.264 video and AAC audio
- **Node server**: `node.polychronis.gr` — WebRTC signaling (WSS), TURN relay, nginx reverse proxy with Bearer token auth
- **iOS app**: SwiftUI, targets iOS 17+, uses `stasel/WebRTC` SPM package (pinned to v140 due to v141 header path regression)
- **Remote access**: autossh reverse tunnel from astrapi to Node (forwards Node:18080 → localhost:8080), nginx proxies `/api/` with Bearer auth
- **WebRTC**: Peer-to-peer when possible, TURN relay through Node when direct connection fails
- **Signaling**: WebSocket-based Node.js signaling server mediates WebRTC negotiation between detection server (aiortc) and iOS (stasel/WebRTC)
- **Docker**: Signaling server + coturn run in Docker containers on Node with `restart: unless-stopped`

## Key Files

### Detection Server (`detection/`)
- `stream_yolo.py` — entry point, parses config sections including `[node]`, starts signaling client
- `detector.py` — core pipeline: FrameBuffer, AudioCapture, EventQueue, ClipRecorder, _transcode, _generate_preview, run()
- `webrtc.py` — aiortc WebRTC: FrameBufferTrack (video), signaling client, `_handle_viewer`, uses `candidate_from_sdp()` for ICE candidates
- `web.py` — Tornado HTTP server: MJPEG stream, review UI, stats, SSE `/events` endpoint, configurable bind host
- `reviews.py` — clip review state management, has `prune()` to clean stale entries
- `settings.toml` — main config (model conf=0.8, recording params, node URLs)
- `settings.local.toml` — secrets (gitignored)
- `templates/review.html` — web review UI
- `deerstop.service` — systemd unit for detection pipeline
- `deerstop-tunnel.service` — autossh reverse tunnel systemd unit

### iOS App (`ios/DeerStop/`)
- `project.yml` — XcodeGen project spec (avoids Xcode duplicate file issues)
- `App/DeerStopApp.swift` — app delegate, AVAudioSession setup (.playback category)
- `Services/WebRTCService.swift` — WebRTC peer connection (currently video-only)
- `Services/SignalingService.swift` — WebSocket signaling for WebRTC (ObservableObject)
- `Services/SSEService.swift` — SSE-based local notifications for new detections
- `Services/APIService.swift` — REST API client for clips, Bearer token auth
- `Views/LiveStreamView.swift` — WebRTC live stream display (holds strong `@State` ref to delegate to prevent dealloc)
- `Views/ClipBrowserView.swift` — clip list (sorted newest-first) with animated preview playback, review checkmarks (44x44pt tap targets)
- `Views/ClipPlayerView.swift` — full clip video player (uses `AVURLAssetHTTPHeaderFieldsKey` for auth headers)
- `Views/SettingsView.swift` — server URL and auth token configuration with persistence
- `Models/Clip.swift` — clip model
- `Info.plist` — bundle keys

### Node Server (`node/`)
- `signaling/server.js` — WebSocket signaling server: streamer/viewer registration, SDP/ICE relay, TURN credential generation
- `signaling/package.json` — dependencies (ws library)
- `signaling/test/signaling.test.mjs` — 11 tests (all passing)
- `docker-compose.yml` — signaling + coturn containers
- `coturn/turnserver.conf.template` — TURN relay config
- `nginx/deerstop.conf` — location blocks for `/signaling` (WebSocket proxy), `/api/` (reverse tunnel proxy with Bearer auth)
- `.env.example` — template for secrets (TURN_SECRET, AUTH_TOKEN, TURN_URL)

## Completed Work

### iOS app creation (from web app conversion)
- Built entire native SwiftUI iOS app from scratch as a remote client
- Restructured repo: moved Python files from root into `detection/` (preserving git history)
- Created Node.js signaling server with WebSocket-based WebRTC negotiation
- Set up Docker containers for signaling + coturn on Node
- Configured nginx reverse proxy with Bearer token auth
- Set up autossh reverse tunnel from edge device to Node
- Implemented WebRTC live streaming (video working, audio pending)
- Implemented clip browsing with animated 3-second MP4 previews (not GIFs)
- Implemented clip playback with auth header passing via AVURLAsset
- Implemented SSE-based local notifications
- Implemented review marking with 44x44pt tap targets
- Fixed multiple bugs: ICE candidate parsing (aiortc needs `candidate_from_sdp()`), StreamDelegate dealloc, AVURLAsset auth headers, SignalingService ObservableObject conformance, Authorization header (Basic → Bearer)

### Detection confidence increased
- `settings.toml`: `conf` changed from 0.6 to 0.8

### Thumbnail removal
- JPG thumbnails were redundant (replaced by animated preview videos)
- Removed all thumbnail logic from: detector.py, Clip.swift, ClipBrowserView.swift, APIService.swift

### PST timezone for clip filenames
- Changed `datetime.now(timezone.utc)` to `datetime.now(ZoneInfo("America/Los_Angeles"))` in ClipRecorder._start()

### Clips sorted newest-first
- ClipBrowserView.swift: sort changed to `$0.id > $1.id`

### Audio recording for clips
- Added `AudioCapture` class in detector.py (~130 lines):
  - Continuous PCM ring buffer from RTSP audio via ffmpeg subprocess
  - Probes RTSP with ffprobe for audio stream before starting
  - Background thread reads 0.5s PCM chunks into deque with wall-clock timestamps
  - `extract(start_time, end_time, out_dir)` writes WAV file for a time range
- Modified `_transcode()` to accept `audio_path` and mux audio with ffmpeg
- Modified `ClipRecorder` to track wall-clock times and extract audio on clip close
- Wired up in `run()`: created for RTSP sources, passed to ClipRecorder, restarts on reconnect

### Audio in preview clips
- Removed `-an` flag from `_generate_preview()`, added `-c:a aac -b:a 64k`
- Unmuted iOS preview player (removed `player.isMuted = true` from PlayerUIView)

### iOS audio playback fix
- Added AVAudioSession `.playback` category setup in DeerStopApp.swift
- Required for AVPlayer to produce sound

## Pending Work

### WebRTC live stream audio (IN PROGRESS - was in plan mode)
- **Problem**: Live page on iOS app has no audio. WebRTC only sends video.
- **Server side** (`webrtc.py`): Only has `FrameBufferTrack` (kind="video"). Needs an audio track.
- **iOS side** (`WebRTCService.swift`): Only handles `RTCVideoTrack`. Audio tracks are auto-played by WebRTC once received, but may need explicit handling.
- **Approach**: Create an `AudioStreamTrack` (aiortc MediaStreamTrack, kind="audio") that reads real-time PCM from the existing AudioCapture's buffer. Add it to the peer connection in `_handle_viewer`. The AudioCapture instance from detector.py can be shared/passed to the signaling client.
- **Key consideration**: AudioCapture already buffers PCM chunks with timestamps. Need to add a real-time streaming method (e.g., queue or event-based) so the WebRTC track can get chunks as they arrive rather than from the ring buffer.

### Push notifications via APNs
- User wants push notifications (currently only has SSE-based local notifications)
- Requires: APNs key from Apple Developer portal, server-side sending from node server
- User said "we'll get back to this"

## Deployment

- SSH to edge device: `ssh astrapi`
- Pull changes: `cd /path/to/deer_stop && git pull`
- Restart service: `sudo systemctl restart deerstop` (requires password, user runs manually)
- Node server: `cd node && docker-compose up -d`
- Firewall on Node: open `443/tcp`, `3478/tcp+udp`, `49152-65535/udp`
- Prune stale reviews after deleting clips: `cd detection && python -c "from reviews import *; prune(...)"`
- Delete stale preview files after code changes: remove old `.preview.mp4` files so they regenerate
- XcodeGen: `cd ios && xcodegen generate` to regenerate Xcode project from `project.yml`

## Git Workflow

- PRs created with `gh pr create`, merged with `gh pr merge`
- Main branch: `main`
- PRs so far: #1 (audio recording), #2 (preview audio), #3 (AVAudioSession fix)
- iOS app initial work was merged directly via `git merge` (before `gh` auth was set up)

## Gotchas

- cv2.VideoCapture/VideoWriter are video-only — audio requires separate ffmpeg processes
- iOS AVPlayer is silent without AVAudioSession `.playback` category
- Preview clips have skip logic (`if preview.exists(): return`) — old previews must be deleted when preview generation changes
- The service on astrapi must be restarted after pulling new code
- `sudo` over SSH requires a terminal — provide commands for user to run manually
- Clips recorded before AudioCapture was deployed have no audio tracks
- aiortc `RTCIceCandidate` takes parsed fields, not raw SDP strings — must use `candidate_from_sdp()`
- `stasel/WebRTC` v141 has a header path regression — pin to v140
- `AVURLAsset` ignores custom HTTP headers passed normally — must use `AVURLAssetHTTPHeaderFieldsKey` option
- iOS WebRTC `StreamDelegate` can be immediately deallocated if not held with `@State`
- Xcode can create duplicate `Services 2/` directories — use XcodeGen (`project.yml`) to avoid this

---
> Source: [ypodim/deer_stop](https://github.com/ypodim/deer_stop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

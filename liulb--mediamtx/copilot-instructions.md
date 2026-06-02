## mediamtx

> This is a self-hosted telemedicine video streaming system that replaces Tencent Cloud TRTC service to achieve 90% cost savings (from ¥7,300/month to ¥700/month). The system enables **bidirectional video and audio communication** between a web-based doctor client and a WeChat mini-program patient client.

# MediaMTX Telemedicine Video System

## Project Overview

This is a self-hosted telemedicine video streaming system that replaces Tencent Cloud TRTC service to achieve 90% cost savings (from ¥7,300/month to ¥700/month). The system enables **bidirectional video and audio communication** between a web-based doctor client and a WeChat mini-program patient client.

**Key Achievement**: Bidirectional video streaming with 3-5 second latency using MediaMTX as the central media server, with FFmpeg-based audio transcoding for cross-platform compatibility.

## Architecture

```
┌─────────────────┐          ┌─────────────────┐          ┌──────────────────┐          ┌─────────────────┐
│   Doctor端      │          │   MediaMTX      │          │   FFmpeg         │          │   患者端        │
│   (Web)         │          │   Server        │          │   Transcoder     │          │  (小程序)       │
├─────────────────┤          ├─────────────────┤          ├──────────────────┤          ├─────────────────┤
│ Push: WebRTC    │─ WHIP ─>│ Port 8889       │          │ Input: RTSP      │          │ Push: RTMP      │
│ (H.264+Opus)    │          │ (Opus audio)    │─ RTSP ─>│ (Opus→AAC)       │─ RTMP ─>│ (H.264+AAC)     │
│                 │          │                 │          │ Output: RTMP     │          │                 │
│ Play: HLS       │<─ HLS ──│ Port 8888       │<─ RTMP ─│ (AAC audio)      │          │ Play: RTMP      │
│ (fMP4)          │          │ (AAC audio)     │          │                  │          │ (AAC audio)     │
└─────────────────┘          └─────────────────┘          └──────────────────┘          └─────────────────┘
                                      ↑                                                            │
                                      └────────────────── RTMP ────────────────────────────────────┘
```

### Technology Stack

- **MediaMTX**: Central streaming media server (bluenviron/mediamtx:latest)
  - Handles WebRTC (WHIP), RTMP, RTSP, HLS protocols
  - Real-time protocol conversion

- **FFmpeg Transcoder**: Audio codec conversion (jrottenberg/ffmpeg:6-alpine)
  - Converts Opus (WebRTC) → AAC (RTMP/iOS compatible)
  - Video passthrough (no re-encoding)

- **Doctor端 (web-doctor/)**:
  - React + Vite
  - WebRTC push via WHIP protocol (H.264 + Opus)
  - HLS playback with hls.js (supports fMP4 + AAC)
  - Forced H.264 encoding via SDP manipulation

- **患者端 (miniapp-patient/)**:
  - WeChat Mini-Program
  - RTMP push via live-pusher component (H.264 + AAC)
  - RTMP playback via live-player component (requires AAC audio)
  - Manual audio playback trigger for iOS

### Port Mapping

| Port | Protocol | Purpose |
|------|----------|---------|
| 1935 | RTMP | Mini-program RTMP push/pull, FFmpeg output |
| 8554 | RTSP | MediaMTX RTSP server, FFmpeg input |
| 8888 | HTTP | HLS playback endpoint |
| 8889 | HTTP | WebRTC/WHIP endpoint |
| 8189 | UDP | WebRTC ICE/media |
| 9997 | HTTP | MediaMTX API (requires auth) |
| 5173 | HTTP | Vite dev server (doctor端) |

## Common Commands

### Start All Services

```bash
cd /document/livekit-demo
docker-compose up -d

# View logs
docker logs -f mediamtx
docker logs -f transcoder
```

### Start Doctor端 (Web Client)

```bash
cd web-doctor
npm run dev  # Runs on http://localhost:5173
```

### Useful Docker Commands

```bash
docker-compose down               # Stop all services
docker-compose restart mediamtx   # Restart MediaMTX
docker-compose restart transcoder # Restart audio transcoder
docker logs mediamtx --tail 50    # View MediaMTX logs
docker logs transcoder --tail 50  # View FFmpeg logs
```

### Testing Endpoints

```bash
# Test HLS streams (in browser)
http://192.168.20.209:8888/doctorStream_aac/index.m3u8  # Doctor stream (AAC audio)
http://192.168.20.209:8888/patientStream/index.m3u8     # Patient stream

# Test RTMP streams (using FFplay)
ffplay rtmp://192.168.20.209:1935/doctorStream_aac
ffplay rtmp://192.168.20.209:1935/patientStream
```

## Key Files and Their Purposes

### Core Configuration

- **config/mediamtx.yml**: MediaMTX server configuration
  - Enables RTMP, RTSP, HLS, WebRTC services
  - HLS settings: `hlsVariant: mpegts`, 3 segments, 1s duration
  - WebRTC ICE configuration with STUN server
  - NAT IP configuration: `webrtcICEHostNAT1To1IPs: ['192.168.20.209']`

- **docker-compose.yml**: Container orchestration
  - MediaMTX container (ports, config mounting)
  - FFmpeg transcoder container (audio conversion)
  - Network mode: host (for inter-container communication)

- **transcode.sh**: FFmpeg transcoding script
  - Reads from RTSP: `rtsp://192.168.20.209:8554/doctorStream`
  - Outputs to RTMP: `rtmp://192.168.20.209:1935/doctorStream_aac`
  - Video: copy (no re-encoding)
  - Audio: Opus → AAC (128 kbps, 44.1 kHz)

### Doctor端 (Web)

- **web-doctor/src/App.jsx**: Main application component
  - Lines 13-84: HLS playback with hls.js (fMP4 + AAC support)
  - Lines 86-256: WebRTC push via WHIP protocol
  - Lines 186-230: SDP manipulation to force H.264 encoding
  - Audio control: `videoElement.muted = false` for playback

- **web-doctor/package.json**: Dependencies
  - hls.js: HLS playback library
  - react, react-dom, vite

### 患者端 (Mini-Program)

- **miniapp-patient/pages/index/index.js**: Main page logic
  - `pushUrl`: `rtmp://192.168.20.209:1935/patientStream`
  - `playUrl`: `rtmp://192.168.20.209:1935/doctorStream_aac` (transcoded)
  - `resumeAudio()`: Manual audio unmute for iOS
  - `manualPlay()`: User-triggered playback

- **miniapp-patient/pages/index/index.wxml**: UI template
  - live-pusher: `mode="live"`, `enable-mic="true"`, `muted="false"`
  - live-player: `mode="live"`, `muted="false"`, `sound-mode="speaker"`
  - "🔊 播放声音" button for iOS audio trigger

## Critical Implementation Details

### 1. Audio Codec Transcoding (Opus → AAC)

**Problem**: WebRTC uses Opus audio, but WeChat mini-program (iOS) requires AAC audio.

**Solution**: FFmpeg transcoder container automatically converts audio codecs.

```bash
# FFmpeg transcoding pipeline
RTSP Input (Opus) → FFmpeg → RTMP Output (AAC)
- Video: copy (no re-encoding)
- Audio: opus → aac (128kbps, 44100Hz)
```

**Files**:
- `docker-compose.yml`: Defines transcoder service
- `transcode.sh`: FFmpeg command script
- `config/mediamtx.yml`: Defines `doctorStream_aac` path

### 2. H.264 Video Codec Enforcement

**Problem**: Browser WebRTC defaults to VP8, but MPEG-TS HLS only supports H.264.

**Solution**: SDP manipulation to prioritize H.264 codec in offer.

**Implementation**: web-doctor/src/App.jsx:186-230

```javascript
// Find H.264 payload types in SDP
const h264Payloads = [];
const otherPayloads = [];
payloadTypes.forEach(pt => {
  if (sdp.match(new RegExp(`a=rtpmap:${pt} H264`, 'i'))) {
    h264Payloads.push(pt);
  } else {
    otherPayloads.push(pt);
  }
});

// Reorder: H.264 first, others after
const newPayloadOrder = [...h264Payloads, ...otherPayloads].join(' ');
```

### 3. HLS Playback with hls.js

**Problem**: Chrome doesn't natively support HLS playback.

**Solution**: Use hls.js library with auto-retry and error recovery.

**Implementation**: web-doctor/src/App.jsx:13-84

```javascript
const hls = new Hls({
  enableWorker: true,
  lowLatencyMode: true,
  backBufferLength: 90,
});

hls.on(Hls.Events.ERROR, (event, data) => {
  if (data.type === Hls.ErrorTypes.NETWORK_ERROR) {
    setTimeout(() => hls.loadSource(url), 3000); // Retry
  } else if (data.type === Hls.ErrorTypes.MEDIA_ERROR) {
    hls.recoverMediaError(); // Recover
  }
});
```

### 4. iOS Audio Playback Trigger

**Problem**: iOS requires user interaction to unmute audio.

**Solution**: Manual "🔊 播放声音" button + auto-unmute after video starts.

**Implementation**: miniapp-patient/pages/index/index.js:89-110

```javascript
// Auto-triggered when video starts
resumeAudio() {
  const playerContext = wx.createLivePlayerContext('player')
  playerContext.mute({ muted: false })
  playerContext.resume()
}

// Manual button
manualPlay() {
  const playerContext = wx.createLivePlayerContext('player')
  playerContext.play()
  playerContext.mute({ muted: false })
}
```

### 5. WebRTC ICE Configuration

**Problem**: WebRTC connections fail in NAT environments.

**Solution**: Configure STUN server and local IP.

**Configuration**: config/mediamtx.yml:20-30

```yaml
webrtc: true
webrtcAddress: :8889
webrtcICEServers:
  - stun:stun.l.google.com:19302
webrtcICEHostNAT1To1IPs: ['192.168.20.209']
webrtcICEUDPMuxAddress: :8189
```

## Common Issues and Solutions

### Issue 1: "No audio on mini-program (iOS)"

**Cause**: iOS auto-mute policy or Opus codec incompatibility
**Solution**:
1. Click "🔊 播放声音" button on mini-program
2. Verify transcoder is running: `docker logs transcoder`
3. Check audio codec: `curl http://192.168.20.209:8888/doctorStream_aac/index.m3u8` should show `mp4a.40.2`

### Issue 2: "ICE connection failed"

**Cause**: Incorrect NAT IP configuration
**Solution**:
1. Update `webrtcICEHostNAT1To1IPs` in mediamtx.yml with correct LAN IP
2. Open UDP port 8189 in firewall
3. Restart MediaMTX: `docker-compose restart mediamtx`

### Issue 3: "Mini-program shows no video"

**Cause**: RTMP stream not available or wrong URL
**Solution**:
1. Verify transcoder is running: `docker ps | grep transcoder`
2. Check FFmpeg logs: `docker logs transcoder --tail 20`
3. Ensure doctor端 is pushing (transcoder needs input)
4. Verify RTMP URL: `rtmp://192.168.20.209:1935/doctorStream_aac`

### Issue 4: "Browser can't play HLS (Chrome)"

**Cause**: Native HLS not supported in Chrome
**Solution**: Uses hls.js library (already implemented in web-doctor/src/App.jsx)

### Issue 5: "FFmpeg exits with 'Broken pipe'"

**Cause**: Output RTSP endpoint not accepting connection
**Solution**: Changed to RTMP output (more compatible with MediaMTX)

### Issue 6: "High CPU usage"

**Cause**: Video re-encoding
**Solution**: FFmpeg uses `-c:v copy` (video passthrough, no encoding)

## Testing Checklist

- [x] MediaMTX container running (`docker ps`)
- [x] Transcoder container running and processing (`docker logs transcoder`)
- [x] Doctor端 can access camera/microphone
- [x] Doctor端 shows "🟢 推流中" status
- [x] Mini-program shows "推流成功" toast
- [x] Doctor端 can see patient video (right side)
- [x] Doctor端 can hear patient audio
- [x] Mini-program can see doctor video
- [x] Mini-program can hear doctor audio (after clicking "播放声音")
- [x] Latency acceptable (3-5 seconds)

## Troubleshooting Commands

```bash
# Check all containers
docker ps

# Check MediaMTX status and logs
docker logs mediamtx --tail 100
docker logs mediamtx | grep -i "error\|warn"

# Check FFmpeg transcoder
docker logs transcoder --tail 50
docker logs transcoder -f  # Follow mode

# Test stream availability
curl -I http://192.168.20.209:8888/doctorStream_aac/index.m3u8
curl -I http://192.168.20.209:8888/patientStream/index.m3u8

# Check if FFmpeg is transcoding
docker exec transcoder ps aux | grep ffmpeg

# Restart everything
docker-compose down
docker-compose up -d
```

## Network Requirements

- **LAN IP**: 192.168.20.209 (hardcoded in configs)
- **Same Network**: All devices must be on same LAN/WiFi
- **Firewall**: Ports 1935, 8554, 8888, 8889, 8189/UDP must be accessible
- **Mini-Program**: Must enable "不校验合法域名" in WeChat DevTools for local development

## Project Evolution Notes

**Original Plan**: Use LiveKit for both sides
**Problem**: LiveKit doesn't integrate with WeChat mini-programs
**Solution 1**: MediaMTX + WebRTC/RTMP
**Problem 2**: VP8 codec not compatible with HLS MPEG-TS
**Solution 2**: Force H.264 via SDP manipulation
**Problem 3**: Opus audio not compatible with iOS/RTMP
**Solution 3**: FFmpeg audio transcoding (Opus → AAC)

**Abandoned Components**:
- LiveKit server (removed from docker-compose.yml)
- LiveKit client library (removed from package.json)
- Backend token service (server/server.js - not used)
- FLV.js (replaced with hls.js)

**Current Architecture**:
- MediaMTX: Protocol hub (WebRTC, RTMP, RTSP, HLS)
- FFmpeg: Audio transcoding layer (Opus → AAC)
- Doctor端: WebRTC push (H.264+Opus) + HLS play (fMP4+AAC)
- Patient端: RTMP push/play (H.264+AAC)

## Cost Comparison

| Service | Monthly Cost | Notes |
|---------|--------------|-------|
| Tencent Cloud TRTC | ¥7,300 | Previous solution |
| Self-hosted MediaMTX + FFmpeg | ¥700 | VPS + bandwidth |
| **Savings** | **¥6,600 (90%)** | Worth the 3-5s latency trade-off |

## Performance Characteristics

| Metric | Value | Notes |
|--------|-------|-------|
| Latency | 3-5 seconds | HLS/RTMP streaming delay |
| CPU Usage | Low | FFmpeg video passthrough (no re-encoding) |
| Audio Transcoding | ~10% CPU | Opus → AAC conversion |
| Bandwidth | ~2 Mbps | Per user (bidirectional) |
| Concurrent Users | 50+ | Limited by VPS resources |

## Future Optimization Paths

1. **Lower Latency**:
   - Switch to WebRTC for mini-program (requires WeChat SDK update)
   - Use LL-HLS (Low-Latency HLS)

2. **Audio Quality**:
   - Increase AAC bitrate (128k → 192k)
   - Use stereo audio throughout

3. **Security**:
   - Add RTMP/WHIP authentication
   - Enable MediaMTX API authentication

4. **Monitoring**:
   - Prometheus + Grafana for stream health
   - Alert on transcoder failures

5. **Production**:
   - HTTPS + domain name + SSL certificates
   - Use public TURN server for firewall traversal

6. **Scale**:
   - Nginx load balancer
   - Multiple MediaMTX instances
   - Horizontal FFmpeg transcoder scaling

7. **Recording**:
   - Enable MediaMTX recording feature
   - Store consultations for compliance

## Documentation Files

- **README.md**: Project overview and quick start
- **CLAUDE.md**: This file - complete system reference (updated 2025-10-16)
- **transcode.sh**: FFmpeg audio transcoding script

## IP Address Changes

If deploying to a different network, search and replace `192.168.20.209` with your new IP in:
- **config/mediamtx.yml**: `webrtcICEHostNAT1To1IPs`
- **transcode.sh**: RTSP input and RTMP output URLs
- **web-doctor/src/App.jsx**: `patientStreamUrl` and `whipUrl`
- **miniapp-patient/pages/index/index.js**: `pushUrl` and `playUrl`

## Quick Start (TL;DR)

```bash
# 1. Start all services (MediaMTX + FFmpeg transcoder)
cd /document/livekit-demo
docker-compose up -d

# 2. Verify transcoder is running
docker logs transcoder

# 3. Start doctor端
cd web-doctor
npm run dev

# 4. Open http://localhost:5173 and click "开始推流"

# 5. Open WeChat DevTools, load miniapp-patient/, compile and test on real device

# 6. On mini-program, click "🔊 播放声音" to enable audio (iOS)

# 7. Both sides should see and hear each other within 15-20 seconds
```

**Expected Result**:
- Doctor sees patient video + hears audio (right side)
- Patient sees doctor video + hears audio (bottom, after clicking audio button)
- Latency: 3-5 seconds
- Video: 640x480 @ 60fps
- Audio: AAC 128kbps stereo

## Troubleshooting Flowchart

```
No audio on mini-program?
├─ Check: Is video showing?
│  ├─ No → Check RTMP URL (should be doctorStream_aac)
│  └─ Yes → Continue
├─ Check: Clicked "播放声音"?
│  ├─ No → Click the button
│  └─ Yes → Continue
├─ Check: docker logs transcoder
│  ├─ FFmpeg running?
│  │  ├─ No → Restart: docker-compose restart transcoder
│  │  └─ Yes → Continue
│  └─ Shows "Stream #0:1: Audio: aac"?
│     ├─ No → Doctor端 not pushing
│     └─ Yes → Check audio codec in m3u8
└─ curl http://192.168.20.209:8888/doctorStream_aac/index.m3u8
   └─ Shows "mp4a.40.2"? → Audio transcoding working
```

## Key Lessons Learned

1. **Codec Compatibility is Critical**: WebRTC (Opus/VP8) vs iOS/RTMP (AAC/H.264)
2. **FFmpeg is Essential**: Browser can't output AAC, iOS can't decode Opus
3. **User Interaction Required**: iOS audio policy requires manual trigger
4. **HLS.js Necessary**: Chrome doesn't support HLS natively
5. **SDP Manipulation Works**: Can force H.264 even if browser prefers VP8
6. **RTMP More Reliable**: Than RTSP for output (fewer broken pipe errors)
7. **Docker Networking**: Host mode simplifies inter-container communication
8. **Auto-Retry Essential**: Streams don't always start immediately

---

**Last Updated**: 2025-10-16
**Status**: ✅ Fully working (bidirectional video + audio)
**Tested On**: Windows 11 (doctor) + iPhone (patient)

---
> Source: [liulb/mediaMTX](https://github.com/liulb/mediaMTX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->

## konnyaku

> > **Origin**: This spec is based on a conversation with Gemini about building a fully local,

# Konnyaku (蒟蒻) — Real-Time Voice Translation POC

> **Origin**: This spec is based on a conversation with Gemini about building a fully local,
> privacy-preserving, real-time voice translation system.

## Goal

Build a **Proof of Concept** for a real-time voice translation system that runs entirely on
local models. The system captures microphone audio via **LiveKit WebRTC**, transcribes it using
**Qwen3-ASR** (pure C implementation), translates the text using **TranslateGemma** (via Ollama),
and displays bilingual subtitles in a web frontend.

## Architecture Overview

The system uses a microservices architecture orchestrated via **Docker Compose**:

1. **LiveKit Server**: Handles WebRTC audio streaming and Data Channels.
2. **Redis**: LiveKit state management.
3. **Ollama**: Runs the `translategemma` model locally for text translation.
4. **Go Agent**: A Go application using CGO. It connects to LiveKit, receives audio tracks,
   decodes Opus to PCM via LiveKit SDK's built-in decoder, performs VAD (Voice Activity Detection),
   runs pure C Qwen3-ASR for transcription, calls Ollama for translation, and publishes results
   back via Data Channel. It also hosts a simple HTTP API on port 8080 to distribute LiveKit tokens.
5. **Web Frontend**: A plain HTML/JS page served by Nginx to capture microphone input and display
   bilingual subtitles. Users can select target translation language dynamically.

## Key Technology Choices

### Qwen3-ASR (antirez/qwen-asr — Pure C implementation)
- **Repo**: https://github.com/antirez/qwen-asr
- Zero dependencies: Pure C, only needs BLAS (Accelerate on macOS, OpenBLAS on Linux)
- Models: 0.6B and 1.7B (0.6B recommended for CPU inference)
- **C API** (actual, from repo):
  ```c
  #include "qwen_asr.h"
  qwen_ctx_t *ctx = qwen_load("qwen3-asr-0.6b");
  // Optional: set callbacks, language, prompt
  qwen_set_token_callback(ctx, my_token_handler, userdata);
  qwen_set_force_language(ctx, "Italian");
  qwen_set_prompt(ctx, "Preserve spelling: PostgreSQL, Redis");
  // Transcribe from file
  char *text = qwen_transcribe(ctx, "audio.wav");
  // Or from raw float samples
  char *text2 = qwen_transcribe_audio(ctx, samples, n_samples);
  free(text);
  qwen_free(ctx);
  ```
- **Build**: `make blas` (not `make lib`)
- Supports streaming via `qwen_transcribe_stream(ctx, samples, n_samples)`
- Can read from stdin with auto-detection (WAV header or raw s16le 16kHz mono)

### TranslateGemma (via Ollama)
- **Blog**: https://blog.google/innovation-and-ai/technology/developers-tools/translategemma/
- Available on Ollama as `translategemma:4b`, `translategemma:12b`, `translategemma:27b`
- 55 language support
- 4B model optimized for mobile/edge, 12B best balance

### LiveKit (Go SDK v2)
- **Repo**: https://github.com/livekit/server-sdk-go
- **SDK**: `github.com/livekit/server-sdk-go/v2`
- **Connection API** (actual, from docs):
  ```go
  room, err := lksdk.ConnectToRoom(hostURL, lksdk.ConnectInfo{
      APIKey:              apiKey,
      APISecret:           apiSecret,
      RoomName:            roomName,
      ParticipantIdentity: identity,
  }, roomCB)
  ```
- **Built-in Opus→PCM16 decoding** via `lkmedia.NewPCMRemoteTrack`:
  ```go
  import lkmedia "github.com/livekit/server-sdk-go/v2/pkg/media"
  pcmTrack, err := lkmedia.NewPCMRemoteTrack(
      remoteTrack, writer,
      lkmedia.WithTargetSampleRate(16000),  // Resample for ASR
      lkmedia.WithTargetChannels(1),
  )
  ```
- **Token creation**:
  ```go
  import "github.com/livekit/protocol/auth"
  at := auth.NewAccessToken(apiKey, apiSecret)
  grant := &auth.VideoGrant{RoomJoin: true, Room: room}
  at.SetVideoGrant(grant).SetIdentity(identity).SetValidFor(time.Hour)
  token, err := at.ToJWT()
  ```

### VAD (Voice Activity Detection)
- Use `github.com/maxhawkins/go-webrtc-vad` for speech/silence detection
- Aggressive mode, 600ms+ silence triggers segment end

## Directory Structure

```
konnyaku/
├── docker-compose.yml
├── livekit.yaml
├── models/                  # Place qwen3-asr model weights here
├── agent/
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   ├── main.go              # LiveKit connection, token API, audio pipeline
│   └── translation.go       # Ollama/TranslateGemma client
└── web/
    └── index.html            # Frontend with language selector & subtitles
```

## Docker Compose Services

| Service    | Image/Build     | Ports          | Purpose                     |
|------------|-----------------|----------------|-----------------------------|
| livekit    | livekit-server  | 7880,7881,7882 | WebRTC server               |
| redis      | redis:alpine    | 6379           | LiveKit state               |
| ollama     | ollama/ollama   | 11434          | Translation model           |
| agent      | ./agent         | 8080           | Go agent + Token API        |
| frontend   | nginx:alpine    | 3000           | Web UI                      |

## Data Flow

```
Browser Mic → LiveKit (WebRTC) → Go Agent
                                     │
                                     ├─ Opus→PCM16 (LiveKit SDK built-in)
                                     ├─ VAD (go-webrtc-vad)
                                     ├─ ASR (qwen-asr CGO, float* samples)
                                     ├─ Translate (Ollama HTTP API)
                                     │
                                     └─ Data Channel → Browser Subtitles
```

## Frontend Features
- Target language selector: English, Traditional Chinese, Simplified Chinese,
  Japanese, Korean, Spanish, French
- Dynamic token acquisition from Agent's `/api/token` endpoint
- Bilingual subtitle rendering (original + translated)
- Auto-scroll subtitle container

## Cross-Platform Notes
- **Docker (Linux/NVIDIA)**: Full GPU support via `deploy.resources.reservations`
- **Docker (CPU)**: Remove GPU reservations, set `MODEL_DEVICE=cpu`
- **Mac (Apple Silicon)**: Docker can't pass GPU; run models bare-metal with MPS
  - ASR: PyTorch with `torch.device("mps")` or keep using C engine (uses Accelerate)
  - Translation: Ollama for Mac (native Metal support) or MLX framework
- **Mac Agent**: `brew install pkg-config opus`, `CGO_ENABLED=1 go run .`

## Execution Instructions

1. Place Qwen3-ASR model weights in `models/` directory
2. Run `go mod tidy` in `agent/` to resolve dependencies
3. Start: `docker-compose up --build -d`
4. Pull TranslateGemma: `docker exec -it <ollama_container> ollama pull translategemma:4b`
5. Open `http://localhost:3000`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianshen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julianshen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

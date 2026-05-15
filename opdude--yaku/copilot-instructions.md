## yaku

> Real-time audio transcription and translation overlay. Captures system audio (loopback), transcribes with whisper.cpp, translates with a local Ollama LLM, and displays results in a semi-transparent Wails desktop overlay window.

# yaku — codebase guide for AI agents

## What this project is

Real-time audio transcription and translation overlay. Captures system audio (loopback), transcribes with whisper.cpp, translates with a local Ollama LLM, and displays results in a semi-transparent Wails desktop overlay window.

## Build system

Use `task` (Taskfile.yml), never `go build` directly — CGO needs `PKG_CONFIG_PATH` set for whisper.cpp, and Wails needs its own build process.

```
task              # build binary (bin/yaku)
task dev          # Wails dev mode with live-reload frontend
task test         # run all tests
task test-verbose # verbose tests
task appimage     # build AppImage
task clean-all    # wipe whisper compiled libs (forces full rebuild)
GPU=cuda task     # build with CUDA instead of Vulkan
```

The `whisper` task compiles whisper.cpp from `third_party/whisper.cpp/` via CMake, installs headers + static libs to `build/whisper/install/`, generates pkg-config files, and is skipped when already built. The `wails-cli` task installs the Wails CLI via `go install` and is also skipped when `wails version` succeeds — both `default` and `dev` depend on it so the user never needs to install Wails manually.

## Key file map

```
internal/
  audio/        Capturer interface (audio.go), IsSilent VAD, PCMToFloat32
                capturer_linux.go   — parec/pactl (production)
                capturer_windows.go — WASAPI stub (TODO Stage 2)
                capturer_darwin.go  — CoreAudio stub (TODO Stage 2)
  config/       Config struct, Load/Save (~/.config/yaku/config.yaml)
  transcribe/   Transcriber wraps sys/whisper CGO bindings; segment callback for partial results
  translate/    Ollama NDJSON streaming; Stream() with token callback, buildPrompt, LangName
cmd/yaku/
  main.go         Wails entry point (window options, asset embed, App binding)
  app.go          Go↔JS bridge — StartPipeline/StopPipeline/GetConfig/SaveConfig
  platform_linux.go / platform_darwin.go / platform_windows.go  — build-tagged window options
  wails.json      Wails project config
  frontend/       HTML/CSS/JS overlay UI
third_party/whisper.cpp/   git submodule; compiled by `task whisper`
build/
  build-appimage.sh   AppImage packaging script
  AppRun              AppImage entry script
  whisper/install/    compiled whisper.cpp headers + static libs
```

## Architecture — pipeline

```
audio.Capturer.Stream()  ──►  500 ms increments channel (never stops)
                                       │
                               rolling audio buffer (10 s)
                                       │
                         (≥ ChunkSeconds of non-silent audio)
                                       │
                          Transcriber.Transcribe()
                           ├─ segment callbacks → runtime.EventsEmit("translation:segment")
                           └─ final text        → runtime.EventsEmit("translation:source")
                                       │
                          translate.Stream()
                           └─ token callbacks   → runtime.EventsEmit("translation:token")
                                                → runtime.EventsEmit("translation:done")
```

Audio capture never stops. Transcription and translation run concurrently with the audio buffer filling in the background.

## CGO dependency

`internal/transcribe` imports `github.com/mutablelogic/go-whisper/sys/whisper` — a direct CGO wrapper around whisper.cpp. This is the ONLY CGO package; the higher-level `pkg/whisper` (requires FFmpeg) is deliberately avoided.

Compilation requires `PKG_CONFIG_PATH` pointing at `build/whisper/install/lib/pkgconfig`.

## Wails frontend

Plain HTML/CSS/JS (no framework) in `cmd/yaku/frontend/`. The Wails runtime injects `window.go` (bound Go methods) and `window.runtime` (events, window management).

Key events the Go backend emits:
- `pipeline:status` `{status}` — listening / transcribing / translating / stopped / error
- `translation:segment` `{text}` — partial whisper segment (live source text)
- `translation:source` `{text}` — final confirmed source text for this chunk
- `translation:token` `{token}` — streaming translation token
- `translation:done` `{source, translation}` — chunk complete

Key Go methods callable from JS:
- `App.StartPipeline()` / `StopPipeline()` / `IsRunning()`
- `App.GetConfig()` → `config.Config`
- `App.SaveConfig(cfg)` → persists to disk
- `App.GetAudioDevices()` → `[]string`

## Window behaviour

- `GDK_BACKEND=x11` set in `platform_linux.go` so `Frameless: true` is respected by Wayland compositors
- `WindowIsTranslucent: false` — opaque window avoids WebKitGTK compositor caching artefacts
- `BackgroundColour: {R:8, G:8, B:14, A:255}` — matches CSS `--overlay-bg`
- `runtime.WindowSetAlwaysOnTop(ctx, true)` called in startup
- Drag: CSS `--wails-draggable: drag` on the header bar

## Tests

`task test` covers audio VAD + PCM conversion, config load/save, transcribe (stub model — no weights), and translate (mock HTTP server). Integration tests with a real model need `WHISPER_MODEL_REAL=<path>`.

---
> Source: [opdude/yaku](https://github.com/opdude/yaku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->

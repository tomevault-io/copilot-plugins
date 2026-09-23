## ystore-go

> This file provides context for AI coding agents working on this project.

# AGENTS.md

This file provides context for AI coding agents working on this project.

## Project Overview

`ystore-go` is a Go tool that hides arbitrary data inside 1920×1080
H.264 video files (and optionally their AAC audio track) in a way that survives
YouTube re-encoding. Data is stored via a visual grid of gray-level cells in
each video frame, protected by Reed-Solomon error correction.

## Architecture

- `cmd/encode/main.go` — CLI entry point for encoding data into video
- `cmd/decode/main.go` — CLI entry point for extracting data from video
- `internal/vcodec/` — Visual grid codec: pack bytes into cell gray levels,
  render frames, read frames back
- `internal/acodec/` — MFSK audio codec: OFDM-like tone modulation,
  Goertzel-based detection, WAV I/O
- `internal/ecc/` — Reed-Solomon error correction (wraps
  `klauspost/reedsolomon`)
- `internal/formats/` — Frame header structs (magic, frame number,
  total frames, data length)
- `internal/pipeline/` — Orchestrates encode/decode end-to-end
  (temp files, ffmpeg invocation, frame sorting)

## Conventions

- Go 1.24+
- Single `go.mod` at root, no vendoring
- All library code under `internal/` — not a public Go library
- CLI flags use long-form (`--input`, not `-i`)
- Error handling: wrap with `fmt.Errorf("context: %w", err)` always
- Test files live alongside their package (`*_test.go` in same dir)
- ffmpeg is the only external runtime dependency; find it via `findFFmpeg()`

## Build & Test

```bash
make build    # builds cmd/encode and cmd/decode
make test     # go test ./...  (requires ffmpeg in PATH)
make lint     # go vet ./...
```

## Key Design Decisions

- Cell size 24px, guard band 4px, 2 bits/cell (4 gray levels): empirically
  survives CRF 18 H.264 without RS errors.
- RS params auto-matched to grid capacity via `MatchRSConfig()`.
- Audio is secondary/low-rate; video frame grid is the primary channel.
- ffmpeg is exec'd, not linked; required at runtime for H.264 encode/decode.

---
> Source: [fabricedge/ystore-go](https://github.com/fabricedge/ystore-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->

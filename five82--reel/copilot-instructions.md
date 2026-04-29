## reel

> This file provides guidance when working with code in this repository.

# AGENTS.md

This file provides guidance when working with code in this repository.

CLAUDE.md and GEMINI.md are symlinks to this file so all agent guidance stays in one place.
Do not modify this header.

## TL;DR

- Do not run `git commit` or `git push` unless explicitly instructed.
- Run `./check-ci.sh` before handing work back.
- Use Context7 MCP for library/API docs without being asked.

## Project Snapshot

Reel is an **AV1 encoding tool** using FFMS2 + SvtAv1EncApp for parallel chunked encoding. It provides opinionated defaults, automatic crop detection, HDR preservation, and post-encode validation.

- **Scope**: Single-developer hobby project - avoid over-engineering
- **Environment**: Go 1.26+, FFmpeg (libopus), SvtAv1EncApp, FFMS2, MediaInfo
- **Design**: Library-first for Spindle embedding, with CLI wrapper

## Related Repos

| Repo | Path | Role |
|------|------|------|
| reel | `~/projects/reel/` | AV1 encoding tool (this repo) |
| spindle | `~/projects/spindle/` | Orchestrator that shells out to Reel during ENCODING |
| flyer | `~/projects/flyer/` | Read-only TUI for Spindle (not a Reel consumer) |

GitHub: [reel](https://github.com/five82/reel) | [spindle](https://github.com/five82/spindle) | [flyer](https://github.com/five82/flyer)

## Build, Test, Lint

```bash
go build -o reel ./cmd/reel       # Build CLI
go test ./...                         # Test
go test -race ./...                   # Race detector
golangci-lint run                     # Lint
./check-ci.sh                         # Full CI (recommended before handoff)
```

## Architecture

```
reel.go, events.go     # Public API: Encoder, Options, EventHandler
cmd/reel/main.go       # CLI wrapper (flag-based)
internal/
├── config/              # Configuration and defaults
├── discovery/           # Video file discovery
├── encoder/             # SVT-AV1 command building
├── encode/              # Parallel chunk encoding pipeline
├── chunk/               # Chunk management
├── keyframe/            # Keyframe extraction
├── worker/              # Worker pool for parallel encoding
├── ffms/                # FFMS2 bindings for frame-accurate indexing
├── ffmpeg/              # FFmpeg parameter building (audio, filters)
├── ffprobe/             # Media analysis
├── mediainfo/           # HDR detection
├── processing/          # Orchestrator, crop detection, audio
├── validation/          # Post-encode validation checks
├── reporter/            # Progress: Terminal, Composite
├── logging/             # File logging setup
└── util/                # Formatting, file utils, system info
```

## Entry Points

| Task | Start Here |
|------|------------|
| Encoding parameters | `internal/config/config.go`, `internal/encoder/encoder.go` |
| Parallel encoding | `internal/encode/encode.go` |
| Keyframe extraction | `internal/keyframe/keyframe.go` |
| Chunk management | `internal/chunk/chunk.go` |
| Crop detection | `internal/processing/crop.go` |
| Validation checks | `internal/validation/validate.go` |
| Terminal output | `internal/reporter/terminal.go` |
| HDR detection | `internal/mediainfo/mediainfo.go`, `internal/ffprobe/ffprobe.go` |
| Public API | `reel.go` |
| CLI flags | `cmd/reel/main.go` |

## CLI Output Style

1. Four sections in human mode: Hardware -> Video -> Encoding -> Validation -> Results
2. Colors via `fatih/color`, progress via `schollz/progressbar`
3. Show progress info once (progress bar during encode, summary after validation)
4. Natural language sentences; emphatic formatting for key values only

## Spindle Integration

See `docs/spindle-integration.md` for library API usage and event types.

## Principles

1. Keep it simple - small hobby project
2. Prefer unit tests over actual encodes (encoding is slow)
3. When running reel with timeout, use at least 120 seconds

## Log Files

Reel logs are stored in `~/.local/state/reel/logs/`

---
> Source: [five82/reel](https://github.com/five82/reel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->

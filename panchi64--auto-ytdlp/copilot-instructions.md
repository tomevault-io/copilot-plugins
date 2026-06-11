## auto-ytdlp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

```bash
cargo build --release       # Build release binary
cargo run --release         # Run TUI mode (default)
cargo run --release -- --auto  # Run automated mode (no TUI)
cargo check                 # Fast compile check
cargo clippy                # Lint
cargo fmt                   # Format
```

## External Dependencies

Requires `yt-dlp` and `ffmpeg` installed and available in PATH. Validated at startup.

## Key Files

- `links.txt`: Download queue (URLs, one per line)
- `download_archive.txt`: yt-dlp archive to prevent re-downloads
- `~/.config/auto-ytdlp/settings.json`: Persistent settings

## Coding Standards

- **No dead code**: Do not use `#[allow(dead_code)]`. If code exists, it must be used.

---
> Source: [panchi64/auto-ytdlp](https://github.com/panchi64/auto-ytdlp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->

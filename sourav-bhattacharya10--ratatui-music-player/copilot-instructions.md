## ratatui-music-player

> A terminal-based music player built with Rust, utilizing `ratatui` for the user interface and `rodio` for audio playback. The application uses `tokio` for asynchronous task management, ensuring a responsive UI while handling audio processing in the background.

# Ratatui Music Player

A terminal-based music player built with Rust, utilizing `ratatui` for the user interface and `rodio` for audio playback. The application uses `tokio` for asynchronous task management, ensuring a responsive UI while handling audio processing in the background.

## Architecture Overview

The project follows a decoupled architecture where the UI and audio engine run in separate tasks and communicate via channels (`tokio::sync::mpsc`).

- **`src/main.rs`**: The entry point. Initializes the terminal, spawns the audio engine with the playlist tracks, and handles the main event loop.
- **`src/player.rs`**: Implements the `audio_engine`. Manages audio playback, auto-advances tracks, and handles commands like `PlayTrack`, `Next`, and `Previous`.
- **`src/state.rs`**: Defines `App`, `PlayerState`, `Playlist`, `Track`, `AudioCommand`, and `AudioEvent`.
- **`src/ui.rs`**: Renders the TUI, including the progress bar, playback instructions, and the interactive playlist.
- **`src/input.rs`**: Handles keyboard and mouse events. Supports track navigation and playlist selection via mouse clicks.

## Key Features

- **Playlist Management**: Supports multiple tracks with auto-advance and dynamic updates.
- **Side-by-Side Layout**: The UI is divided into two main sections:
    - **File Browser (40% width)**: Browse and add music files.
    - **Playlist (60% width)**: View and manage your current tracks.
- **Interactive Focus**:
    - Use `TAB` to switch keyboard focus between the File Browser and the Playlist.
    - The focused section is highlighted with a yellow border.
    - Mouse clicks on a section will automatically switch focus to it.
- **Keyboard Shortcuts**:
    - `TAB`: Toggle focus between File Browser and Playlist
    - `p`, `l`, `r`, `n`, `b`: Playback controls (when Playlist is focused)
    - `Up/Down`, `Enter`, `Backspace`, `a`: Navigation and adding tracks (when File Browser is focused)

## TODOs / Known Limitations

- [ ] Add support for more audio formats.
- [ ] Improve error handling for missing files or audio device failures.
- [ ] Add volume control.
- [ ] Implement playlist saving/loading.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sourav-Bhattacharya10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sourav-Bhattacharya10)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## lofigirl

> This project is a Textual TUI (Terminal User Interface) that acts as a wrapper for `mpv` to play YouTube streams in the terminal. It provides a simple interface for selecting from a list of preset streams.

# Project Overview

This project is a Textual TUI (Terminal User Interface) that acts as a wrapper for `mpv` to play YouTube streams in the terminal. It provides a simple interface for selecting from a list of preset streams.

The application uses the `textual` library to create the TUI and `yt-dlp` to get the direct stream URLs from YouTube. The video is rendered in the terminal using `mpv` with the `--vo=tct` flag.

# Running the Application

## 1. Prerequisites

You will need to have the following installed on your system:

*   Python 3
*   `mpv`

You can install the necessary Python packages using the provided `requirements.txt` file:

```bash
pip install -r requirements.txt
```

## 2. Running the script

Once the dependencies are installed, you can run the application with the following command:

```bash
python lofigirl.py
```

This will launch the TUI. You can then select a preset to start playing the stream.

## 3. Controls

*   Use the arrow keys or tab to navigate the presets.
*   Press `Enter` to select a preset.
*   Press `d` to toggle dark mode.
*   Press `q` or `Ctrl+C` to quit the TUI.
*   While `mpv` is running, you can use its standard keyboard shortcuts (e.g., `q` to quit the player and return to the TUI).

# Development

*   The TUI is built using the `textual` library.
*   The layout and styling are defined in `lofigirl.css`.
*   The presets are defined in the `PRESETS` list in `lofigirl.py`.

---
> Source: [bprendie/lofigirl](https://github.com/bprendie/lofigirl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->

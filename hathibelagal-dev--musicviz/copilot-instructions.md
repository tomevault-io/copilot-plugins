## musicviz

> MusicViz is a Python-based command-line tool designed to generate dynamic music visualizer videos from audio files (MP3 or WAV). It produces high-quality 1920x1080 MP4 videos with animated frequency spectrum bars synchronized to the audio.

# MusicViz Project Context

MusicViz is a Python-based command-line tool designed to generate dynamic music visualizer videos from audio files (MP3 or WAV). It produces high-quality 1920x1080 MP4 videos with animated frequency spectrum bars synchronized to the audio.

## Project Overview

- **Core Functionality**: Converts audio files into visualizer videos using FFT-based frequency analysis.
- **Main Technologies**:
    - **Python 3.11+**: The primary programming language.
    - **librosa**: Used for audio loading and frequency analysis (spectrogram).
    - **matplotlib**: Used to render individual frames of the visualizer.
    - **moviepy**: Handles video assembly and audio embedding.
    - **numpy**: Used for numerical computations and FFT.
    - **FFmpeg**: External system dependency required for video encoding.

## Architecture

The project follows a standard Python package structure:
- `setup.py`: Configuration for packaging and dependencies.
- `src/musicviz/`: Main package directory.
    - `main.py`: Contains the CLI logic, audio processing (`load_audio`), frame generation (`make_frame`), and video creation (`create_visualizer`).

## Building and Running

### System Dependencies
- **FFmpeg**: Must be installed and available in the system PATH.
    - macOS: `brew install ffmpeg`
    - Ubuntu: `sudo apt-get install ffmpeg`

### Installation
From the project root:
```bash
pip install .
```

### Running the Tool
Use the installed `musicviz` command:
```bash
musicviz <input_audio> <output_video> "Video Title"
```

For development, you can run the script directly:
```bash
python -m musicviz.main <input_audio> <output_video> "Video Title"
```

## Development Conventions

- **Environment**: Python 3.11 or 3.12 is recommended.
- **Coding Style**:
    - Uses `argparse` for CLI argument handling.
    - `matplotlib` is used in non-interactive mode (`matplotlib.use('Agg')`).
    - Frames are rendered at 30 FPS.
- **Visuals**:
    - Background: Black.
    - Colormap: `plasma`.
    - Resolution: 1920x1080.
- **Entry Point**: Defined in `setup.py` as `musicviz=musicviz.main:main`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hathibelagal-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

## kokoro-tts

> A high-quality Text-to-Speech (TTS) application built with [Gradio](https://gradio.app/) and powered by the [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) model.

# Kokoro TTS Web UI & CLI

A high-quality Text-to-Speech (TTS) application built with [Gradio](https://gradio.app/) and powered by the [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) model.

## Project Overview

This project provides a user-friendly interface and a command-line tool for generating and streaming speech from text. It leverages the `kokoro` library for fast, high-quality synthesis with support for multiple voices and languages.

### Key Features
- **Web UI**: Interactive Gradio interface for real-time synthesis and streaming.
- **CLI Mode**: Batch process text files into audio chapters from the command line.
- **Batch Export**: Automatically split long texts (like books) into chapters and export as WAV/MP3 files. Supports resuming interrupted exports.
- **Persistent Settings**: Remembers your selected voice across browser sessions using `BrowserState`.
- **Custom Pronunciation**: Support for Markdown link syntax for phonemes (e.g., `[Kokoro](/kˈOkəɹO/)`) and a custom replacement dictionary.
- **Text Normalization**: Smart handling of years, abbreviations, and auto-skipping of reference/bibliography sections.
- **Hardware Acceleration**: Automatic detection and support for CPU, CUDA-enabled GPUs, and Apple Silicon (MPS).

### Key Technologies
- **Python**: Core logic and scripting.
- **Gradio**: Web interface and streaming.
- **Kokoro**: TTS engine (KModel and KPipeline).
- **PyTorch**: Underlying deep learning framework.
- **ZeroGPU (via `spaces`)**: Optional GPU acceleration for Hugging Face Spaces.

## Project Structure

- `app.py`: Main entry point. Supports launching the Web UI or CLI.
- `cli.py`: Logic for the Command-Line Interface.
- `ui/`:
  - `app.py`: Gradio UI definition and frontend logic.
- `core/`:
  - `engine.py`: TTS engine with lazy model loading, hardware auto-detection, and GPU-to-CPU fallback logic.
  - `text.py`: Text normalization, chapter splitting, and reference removal.
- `en.txt`: A collection of random quotes for the "Random Quote" feature.
- `requirements.txt`: Project dependencies.
- `.venv/`: Python virtual environment.

## Building and Running

### Prerequisites
- Python 3.12+
- [ffmpeg](https://ffmpeg.org/) (required for MP3 export and audio processing)

### Installation
1. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Configuration
The application can be configured using environment variables:
- `KOKORO_PORT`: The port on which the Gradio server runs (default: `40001`).
- `KOKORO_HOST`: The host to bind to (default: `0.0.0.0`).
- `KOKORO_API_OPEN`: Whether to keep the API open (default: `True`).

### Running the App

#### Web UI Mode
Execute the main script without arguments to start the Gradio server:
```bash
python app.py
```
Access the UI at `http://localhost:40001` (or your configured port).

#### CLI Mode
Use the `--input` flag to process a text file into audio chapters. By default, audio is exported to `/mnt/c/Users/DavidEnglish/Documents/Kokoro_Exports`.
```bash
python app.py --input my_book.txt --voice af_heart --speed 1.0
```

## Development Conventions

- **Lazy Loading**: Models and voices are loaded on demand during generation to minimize memory footprint and improve startup time.
- **Hardware Detection**: The app automatically selects `cuda`, `mps`, or `cpu`. If a GPU error occurs during generation, it gracefully falls back to CPU.
- **Export Naming**: Combined audio exports use the base filename of the input (e.g., `input.wav`) for better organization.
- **Language Support**: Uses code `'a'` for American English and `'b'` for British English.
- **Chapter Splitting**: Uses regex (default: `^Chapter\s+\d+`) to identify chapter breaks in text files.
- **Streaming**: Supports real-time audio streaming in the Web UI.

## TODO / Future Improvements
- [ ] Add unit tests for synthesis pipelines and text normalization.
- [ ] Implement multi-language support beyond US/UK English.
- [x] Add support for more audio formats (MP3).
- [x] Add resume support for batch exports.
- [x] Add persistent UI settings.

---
> Source: [denglish-w/kokoro-tts](https://github.com/denglish-w/kokoro-tts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

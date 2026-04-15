## njmtech-yt-transcribe

> This project is a Command Line Interface (CLI) tool written in Go that downloads audio from various video platforms like YouTube and Instagram. It checks for the presence of `yt-dlp` and `ffmpeg` and prompts the user to install them if they are missing.

# Project: yt-transcribe - Video Transcriber CLI

## Project Overview

This project is a Command Line Interface (CLI) tool written in Go that downloads audio from various video platforms like YouTube and Instagram. It checks for the presence of `yt-dlp` and `ffmpeg` and prompts the user to install them if they are missing.

## Main Technologies and Architecture

*   **Go (Golang):** The core programming language for the CLI tool.
*   **`flag` package:** Utilized for parsing command-line arguments, allowing users to specify the video URL and an optional output directory.
*   **`yt-dlp` (External Tool):** An essential dependency, `yt-dlp` is an external command-line program responsible for reliably downloading and extracting the audio track from videos on various platforms. The Go application executes `yt-dlp` as a subprocess.
*   **whisper.cpp (External Tool):** The project uses `whisper.cpp` for audio transcription. The Go application executes `whisper-cli` as a subprocess.

The project follows a modular architecture based on SOLID principles:
*   The `main` package orchestrates the overall workflow, handles CLI input, and performs dependency injection.
*   The `pkg/downloader` package defines the `VideoDownloader` interface and provides a concrete `YTDLPAudioDownloader` implementation that interfaces with `yt-dlp`.
*   The `pkg/transcriber` package defines the `Transcriber` interface and includes a `WhisperCPPTranscriber` that interfaces with the `whisper-cli` command.

This design promotes loose coupling, making it straightforward to swap out different audio downloading mechanisms or transcription services without altering the core logic.

## Building and Running

To build and run this CLI tool, follow these steps:

1.  **Install Go:**
    Ensure you have Go version `1.25.5` or newer installed on your system. You can download it from [https://golang.org/doc/install](https://golang.org/doc/install).

2.  **Install `yt-dlp`:**
    The `yt-dlp` command-line tool is a prerequisite for audio downloading. The application explicitly checks for its presence. Install it and ensure it's accessible in your system's PATH.
    *   **Linux/macOS Example:**
        ```bash
        sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
        sudo chmod a+rx /usr/local/bin/yt-dlp # Give execute permissions
        ```
    *   **Windows:** Download `yt-dlp.exe` from [https://github.com/yt-dlp/yt-dlp/releases](https://github.com/yt-dlp/yt-dlp/releases) and add its directory to your system's PATH.

3.  **Install `ffmpeg`:**
    `ffmpeg` is required by `yt-dlp` for post-processing audio, such as converting to WAV format. The application explicitly checks for its presence. Install it and ensure it's accessible in your system's PATH.
    *   **Linux (Debian/Ubuntu) Example:**
        ```bash
        sudo apt update
        sudo apt install ffmpeg
        ```
    *   **macOS (using Homebrew):**
        ```bash
        brew install ffmpeg
        ```
    *   **Windows:** Download `ffmpeg` from [https://ffmpeg.org/download.html](https://ffmpeg.org/download.html) and add its binaries directory to your system's PATH.

4.  **Install and configure `whisper.cpp`:**
    The `whisper-cli` command-line tool from the `whisper.cpp` project is required for transcription.
    *   **Installation:** Follow the instructions at [https://github.com/ggerganov/whisper.cpp](https://github.com/ggerganov/whisper.cpp) to build `whisper.cpp` and get the `whisper-cli` executable.
    *   **Model:** Download a `whisper.cpp` model (e.g., `ggml-base.en.bin`) and place it in a known directory.
    *   **Environment Variable:** Set the `WHISPER_MODEL_PATH` environment variable to the full path of your downloaded model. You can do this by creating a `.env` file in the project root:
        ```
        WHISPER_MODEL_PATH="/path/to/your/ggml-model.bin"
        ```

5.  **Build the CLI Tool:**
    Navigate to the project's root directory (`/home/user/dev/github/projects/njmtech-yt-transcribe`) in your terminal and compile the application:
    ```bash
    go build -o yt-transcribe
    ```
    This will create an executable named `yt-transcribe` (or `yt-transcribe.exe` on Windows) in the current directory.

6.  **Run the Tool:**
    Execute the tool by providing a video URL using the `-url` flag. You can also specify a custom output directory with the `-output` flag.
    ```bash
    ./yt-transcribe -url <VIDEO_URL> [-output <OUTPUT_DIRECTORY>]
    ```
    *   **Example Usage:**
        ```bash
        ./yt-transcribe -url https://www.youtube.com/watch?v=dQw4w9WgXcQ
        ```
    *   The downloaded audio will be saved as a `.wav` file. By default, it uses your system's temporary directory for output.

## Development Conventions

*   **Go Standard Formatting:** The codebase adheres to standard Go formatting practices, ensuring readability and consistency.
*   **SOLID Principles:** The design heavily utilizes interfaces (`VideoDownloader`, `Transcriber`) to promote modularity, extensibility, and testability, in line with SOLID principles (e.g., Dependency Inversion, Interface Segregation).
*   **Clear Comments:** Functions, interfaces, and complex logic sections are well-commented, explaining their purpose, usage, and any external dependencies or conceptual aspects (e.g., the mock transcriber).
*   **Robust Error Handling:** Errors are managed using Go's idiomatic error return values, and critical failures are handled with `log.Fatalf`.
*   **Temporary File Management:** Downloaded audio files are treated as temporary and are automatically cleaned up after transcription.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omoinjm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/omoinjm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

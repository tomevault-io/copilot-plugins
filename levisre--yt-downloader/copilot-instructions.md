## yt-downloader

> This is a C++ desktop application for downloading YouTube videos. It uses GTK4 for the graphical user interface. The application allows users to input multiple YouTube video URLs, choose a download location, and download the videos. It relies on the command-line tool `yt-dlp` to handle the actual downloading process.

### Project Overview

This is a C++ desktop application for downloading YouTube videos. It uses GTK4 for the graphical user interface. The application allows users to input multiple YouTube video URLs, choose a download location, and download the videos. It relies on the command-line tool `yt-dlp` to handle the actual downloading process.

### Core Technologies

*   **Language:** C++17
*   **GUI Toolkit:** GTK4
*   **Build System:** CMake
*   **Dependencies:**
    *   `libcurl`: (Though listed in `CMakeLists.txt`, it's not directly used in `main.cpp`. `yt-dlp` handles the downloading).
    *   `yt-dlp`: The core video downloading engine.

### Project Structure

*   `main.cpp`: The main entry point of the application. Contains all the logic for the GUI, event handling, and download process.
*   `CMakeLists.txt`: The build script for CMake. It defines the project, finds dependencies, and sets up the build process.
*   `README.md`: The user-facing documentation with instructions on how to build and use the application.
*   `.gitignore`: Specifies files and directories to be ignored by Git.
*   `config.ini`: (Generated at runtime) Stores the last used download folder.

### Key Components & Logic

*   **`main()` function:** The entry point of the application. It finds the `yt-dlp` executable and starts the GTK application.
*   **`activate()` function:** This function is called when the GTK application is started. It creates the main window, all the UI widgets (URL input, folder selection, download button, progress bar), and connects the signals (e.g., button clicks) to their respective handler functions. It also checks if `yt-dlp` was found and shows an error if not.
*   **`on_download_clicked()`:** This is the event handler for the "Download" button. It reads the URLs from the text view, validates them, and if they are valid, it creates a new thread to handle the download.
*   **`download_thread_func()`:** This function runs in a separate thread to avoid freezing the UI. It constructs the `yt-dlp` command and executes it using `popen()`. It reads the output from `yt-dlp` to get the download progress and updates the progress bar on the main thread.
*   **`find_yt_dlp()`:** This utility function searches for the `yt-dlp` executable. It first looks in the current working directory and then searches through the directories listed in the `PATH` environment variable.
*   **Configuration:** The application saves the last selected download folder in `config.ini` and loads it on startup. This is handled by `save_config()` and `load_config()`.

### Build & Run

1.  **Prerequisites:** Make sure you have a C++17 compiler, CMake, GTK4, libcurl, and `yt-dlp` installed.
2.  **Build:**
    ```bash
    mkdir build
    cd build
    cmake ..
    make
    ```
3.  **Run:**
    ```bash
    ./yt_downloader
    ```

### Future Development / Vibe

This project is a good base for a more feature-rich YouTube downloader. Here are some ideas for extending it:

*   **Video/Audio Format Selection:** Add options to choose the video quality and format (e.g., MP4, WebM) or to download only the audio (e.g., MP3).
*   **Download Queue:** Implement a queue to show the status of each individual download when multiple URLs are provided.
*   **Thumbnail Preview:** Show video thumbnails next to the URLs.
*   **Improved Error Handling:** Provide more specific error messages from `yt-dlp` (e.g., video not available, invalid URL).
*   **Playlist Support:** Detect YouTube playlist URLs and offer to download the entire playlist.
*   **UI/UX Polish:** Improve the visual design with CSS and more advanced GTK widgets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/levisre)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/levisre)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

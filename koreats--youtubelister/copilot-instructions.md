## youtubelister

> This project is a web-based YouTube video transcriber. It allows users to input a YouTube channel URL, fetch a list of videos from that channel, and then select videos to transcribe their audio content into text.

# Project Overview

This project is a web-based YouTube video transcriber. It allows users to input a YouTube channel URL, fetch a list of videos from that channel, and then select videos to transcribe their audio content into text.

The application is built with the following technologies:

*   **Backend:** Python with Flask
*   **Frontend:** HTML with Tailwind CSS and vanilla JavaScript
*   **YouTube Integration:** Google API Python Client
*   **Audio Transcription:** OpenAI's Whisper model
*   **Video Downloading:** yt-dlp

The application provides a simple web interface where users can:

1.  Enter a YouTube channel URL.
2.  Fetch a list of videos from the channel.
3.  Select the desired videos for transcription.
4.  Choose the transcription quality/speed.
5.  View the transcription results.
6.  Copy or download the transcriptions in Markdown format.

# Building and Running

## Installation

To run this project, you need to have Python and pip installed.

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3.  **Install the dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

## Running the Application

Once the dependencies are installed, you can start the Flask web server:

```bash
python app.py
```

The application will be available at `http://127.0.0.1:8080`.

# Development Conventions

*   **Backend:** The backend is a single Flask application in `app.py`. It uses the Google API client to interact with the YouTube API and the `yt-dlp` and `whisper` libraries for transcription.
*   **Frontend:** The frontend is a single HTML file (`templates/index.html`) that uses Tailwind CSS for styling and vanilla JavaScript for client-side logic.
*   **API Key:** The YouTube API key is hardcoded in `templates/index.html`. For development, this is acceptable, but for production, it should be moved to a more secure location.
*   **Error Handling:** The application has basic error handling for API requests and transcription processes.
*   **Concurrency:** The application supports both sequential and parallel transcription processing. The parallel mode uses `concurrent.futures.ProcessPoolExecutor` to speed up the transcription of multiple videos.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koreats)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/koreats)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

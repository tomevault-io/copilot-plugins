## live-transcription

> This project implements a real-time audio transcription system with a Go backend and a JavaScript frontend. It captures audio from the browser, streams it via WebSocket, and displays live transcriptions.

# Project Overview: Live Audio Transcription System

This project implements a real-time audio transcription system with a Go backend and a JavaScript frontend. It captures audio from the browser, streams it via WebSocket, and displays live transcriptions.

## Key Components:
- **Backend (`main.go`):** A Go HTTP server with WebSocket support that handles audio streaming and mock transcription.
- **Frontend (`live_transcription_ui.html`, `live_audio_recorder.js`):** A web-based user interface for audio capture, visualization, and displaying real-time transcriptions.

## Development Directives:

### 1. Building and Running the Backend:
To build and run the Go backend, use the following commands:
```bash
go build -o live_transcription main.go
./live_transcription
```
**Important:** The backend runs as a daemon and will not return. To terminate it properly, you must manually stop the process (e.g., using `Ctrl+C` in the terminal where it's running, or `kill <PID>` if run in the background).

### 2. Testing and Compilation:
Any modifications to the codebase **must** be tested to ensure the code compiles and functions as expected. Before committing any changes, always run the build command for the Go backend to verify compilation:
```bash
go build -o live_transcription main.go
```
For frontend changes, ensure the `live_transcription_ui.html` and `live_audio_recorder.js` files load and function correctly in a web browser.

### 3. Library Versions:
Always use the latest versions of the libraries. Before building or running, update Go modules to their latest compatible versions:
```bash
go get -u ./...
```
This ensures that the project benefits from the latest features, bug fixes, and security updates.

### 4. Environment Variables:
- `SUMMARY_PROMPT`: (Optional) Sets the prompt used for summarization by the Vertex AI model. If not set, a default prompt will be used: "Provide a complete summary of the following text, highlighting the key elements:"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/owulveryck)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/owulveryck)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

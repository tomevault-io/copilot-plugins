## vsum

> YouTube Summarizer (vSum) is a Chrome extension that integrates directly into the YouTube interface to provide AI-powered summaries of videos. It leverages a Node.js backend for heavy processing, including audio downloading, transcription, and summarization.

# YouTube Summarizer (vSum) - Project Context

## Project Overview

YouTube Summarizer (vSum) is a Chrome extension that integrates directly into the YouTube interface to provide AI-powered summaries of videos. It leverages a Node.js backend for heavy processing, including audio downloading, transcription, and summarization.

### Key Features

- **Stateless Backend:** Uses "Bring Your Own Key" (BYOK) for Gemini and AssemblyAI.
- **Turbo Processing:** Utilizes AssemblyAI Universal-3 Pro for high-precision transcription and Google Gemini for summarization.
- **Smart Caching:** Transcripts and audio files are cached locally for faster re-analysis.
- **Deep YouTube Integration:** Injects buttons directly into the YouTube masthead and video player.
- **Audio Player:** Includes an embedded audio player in the transcript view for quick review.

## Architecture

### Backend (`/backend`)

A Node.js Express server that orchestrates the data pipeline:

1. **Extraction:** Downloads high-quality audio using `yt-dlp.exe`.
2. **Transcription:** Uses `AssemblyAI` (Universal-3 Pro) for multi-language speech-to-text.
3. **Summarization:** Uses `Google Generative AI` (Gemini) to generate JSON-formatted summaries.
4. **Caching:** Stores audio files in `backend/audio_cache/`.

### Extension (`/extension`)

A Manifest V3 Chrome extension:

- **Content Script (`content.js`):** Injects the UI (buttons and overlay) into YouTube pages.
- **Background Script (`background.js`):** Acts as a message bridge between the content script and the backend API.
- **UI:** Custom HTML/CSS overlay for displaying summaries and transcripts.
- **Storage:** Uses `chrome.storage.local` to cache summaries and store API keys.

## Development Workflow

### Prerequisites

- Node.js installed.
- API Keys for **Google Gemini** and **AssemblyAI**.

### Building and Running

#### 1. Backend Setup

```bash
cd backend
npm install
# Binaries (yt-dlp, ffmpeg) are automatically downloaded during postinstall via setup-bins.js
npm start
```

The backend runs on `http://localhost:5000`.

#### 2. Extension Setup

1. Open `chrome://extensions/` in Chrome.
2. Enable **Developer Mode**.
3. Click **Load unpacked** and select the `extension/` folder.
4. Configure API keys via the extension popup or options page.

### Code Quality & Standards

- **ESLint:** Enforces coding standards across the project. Run with `npm run lint`.
- **Prettier:** Automatically formats code. Run with `npm run format`.
- **Husky & lint-staged:** Automatically runs linting and formatting on every commit.

## Key Files

- `backend/server.js`: Main API implementation.
- `backend/setup-bins.js`: Dependency management for external binaries.
- `extension/manifest.json`: Extension permissions and entry points.
- `extension/content.js`: Main UI injection and state logic.
- `extension/styles.css`: Custom styling for the YouTube overlay.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Trivolous)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Trivolous)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

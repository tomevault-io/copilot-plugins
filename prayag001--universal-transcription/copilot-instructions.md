## universal-transcription

> >


# Universal Transcriptor

Generate complete, verbatim transcripts for YouTube videos using Google Gemini 1.5 Flash.

## ⚠️ WHEN TO USE
- When the user asks to transcribe a YouTube video
- When the user wants the full text content from a video
- When the user provides a YouTube URL and asks for a transcript

## How to Use

### Step 1: Get the YouTube URL
Identify the YouTube URL from the user's request (e.g., `https://youtu.be/...` or `https://www.youtube.com/watch?v=...`)

### Step 2: Run the script
```bash
python "{baseDir}/scripts/universal_transcript.py" "<YOUTUBE_URL>"
```

Example:
```bash
python "{baseDir}/scripts/universal_transcript.py" "https://youtu.be/dQw4w9WgXcQ"
```

### Step 3: Present the results
The script will:
1. Generate a complete verbatim transcript
2. Save it to `~/.openclaw/workspace/out/<video-title>.txt`
3. Print the file path and a preview
4. Show the first 15 lines as a preview

Present the saved file path and key content to the user.

## Output Format
- Verbatim transcript with topic segmentation
- Timestamps at topic changes (format: `### [MM:SS] Topic: <description>`)
- Vertical formatting (short lines for readability)
- Includes metadata header (title, URL, date)
- Saved to `workspace/out/` directory

## Requirements
- `GEMINI_API_KEY` must be set in environment or in `~/.openclaw/.env`
- Python 3 installed and accessible

---
> Source: [prayag001/Universal-Transcription](https://github.com/prayag001/Universal-Transcription) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->

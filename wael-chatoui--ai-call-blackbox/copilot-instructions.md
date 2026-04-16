## ai-call-blackbox

> Screen AI Assistant - A Python voice-controlled AI that captures the screen and answers questions using the Blackbox API.

# CLAUDE.md

## Project Overview

Screen AI Assistant - A Python voice-controlled AI that captures the screen and answers questions using the Blackbox API.

## Architecture

```
User speaks → VoiceInput → ScreenCapture → BlackboxAPI → TextToSpeech → Audio response
```

## File Structure

- `main.py` - Entry point, main application loop
- `voice_input.py` - Speech recognition using Google Speech API
- `screen_capture.py` - Screen capture using mss, outputs base64
- `blackbox_api.py` - HTTP client for Blackbox AI API
- `text_to_speech.py` - TTS using pyttsx3
- `config.py` - Loads API key from .env file

## Key Dependencies

- `SpeechRecognition` + `PyAudio` - Voice input
- `mss` + `Pillow` - Screen capture
- `pyttsx3` - Text-to-speech
- `requests` - HTTP requests
- `python-dotenv` - Environment variables

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python main.py

# Linux: Install system dependencies
sudo apt install portaudio19-dev python3-pyaudio espeak
```

## Configuration

The API key is loaded from `.env` file:
```
BLACKBOX_API_KEY=your_key_here
```

## Code Patterns

- Each module exposes both a class-based interface and a simple function interface
- Screen capture returns base64-encoded PNG
- Blackbox API expects multimodal messages with image_url type for images
- Voice input uses Google Speech Recognition (free, no API key needed)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wael-chatoui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

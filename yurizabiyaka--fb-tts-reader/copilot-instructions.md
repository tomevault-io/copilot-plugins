## fb-tts-reader

> A monorepository containing a customizable Android FB2 reader with TTS capabilities and a Python-based TTS server using XTTS v2 model.

# FB2 Reader with Custom TTS - Project Overview

## Project Vision
A monorepository containing a customizable Android FB2 reader with TTS capabilities and a Python-based TTS server using XTTS v2 model.

## Repository Structure
/
├── tts_server/          # Python TTS backend
└── fb2_reader_android/  # Kivy-based Android app

## TTS Server (`tts_server/`)

### Tech Stack
- **Framework**: Python with uvicorn (ASGI server)
- **TTS Model**: XTTS v2 from coqui-tts
- **Database**: MySQL in Docker
- **Platform**: Linux

### Features & Requirements
- **Authentication**: API key-based auth system
- **Billing**: API key usage tracking and billing
- **Session Management**: Expirable session keys

### API Endpoints
1. **Auth Endpoint**
   - Input: API key
   - Output: Session key (expirable)
   - Purpose: Authenticate and get temporary session

2. **TTS Endpoint**
   - Input: session key, language, speaker, text
   - Output: Generated audio
   - Purpose: Text-to-speech conversion

### Infrastructure
- MySQL database running in Docker container
- Keep API simple and focused on these two endpoints only

## FB2 Reader Android (`fb2_reader_android/`)

### Tech Stack
- **Framework**: Kivy
- **Build Tool**: Buildozer
- **Platform**: Android

### Core Reading Features
- Display FB2 book text
- **Navigation methods**:
  - Swipe up/down to turn pages
  - Horizontal thin scrollbar
  - Direct page number input

### TTS Integration Features
- **Configuration**: Menu to set TTS server URL and API key
- **Auto-playback**: Pages turn automatically as reading progresses
- **Smart text highlighting** with 4 distinct states:
  1. Fragment currently being read aloud (color 1)
  2. Fragments generated and ready to play (color 2)
  3. Text sent to TTS but not yet generated (color 3)
  4. Text not yet sent to TTS (color 4)

### User Controls
- **Continuous navigation**: Can turn pages manually while TTS is playing
- **Return button**: Floating button to jump back to currently spoken fragment
- **Playback control**:
  - Stop playback at any time
  - Long-press on text to select new starting point
  - Floating "speak" button appears after long-press selection

## Development Priorities
1. Keep TTS server API minimal and focused
2. Ensure smooth UI experience in Kivy despite TTS operations
3. Handle async operations properly (TTS generation + playback + UI updates)
4. Maintain reading position state across page turns during playback

## Key Technical Challenges to Consider
- Buffering strategy for TTS audio (how many fragments to pre-generate)
- Synchronization between audio playback position and text highlighting
- Session key refresh logic
- Handling network interruptions during TTS streaming
- Managing Kivy UI thread vs background TTS operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yurizabiyaka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yurizabiyaka)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

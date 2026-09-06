## pod-ai

> iOS podcast player with AI-powered voice interaction, built with SwiftUI.

# Pod AI

iOS podcast player with AI-powered voice interaction, built with SwiftUI.

## What It Does

- Search and play any podcast via iTunes Search API
- Real-time voice conversations with AI about episode content using OpenAI's Realtime API

## Key Features

- **Podcast Search**: iTunes Search API to find any podcast
- **Podcast Playback**: Full audio player with seek, skip (±15/30s), lock screen controls
- **Auto-Transcription**: YouTube transcripts (free) with Whisper API fallback
- **Voice Chat**: Tap mic button to ask questions about podcast content via voice (GPT-4o Realtime)
- **Voice Commands**: Say "go back to the podcast", "skip forward", "rewind" - AI executes via function calling
- **Echo Cancellation**: Single AVAudioEngine with voice processing for clean audio
- **Interruption Support**: Speak while AI is talking to interrupt and ask follow-up
- **Modern UI**: Dark theme, mini player, Now Playing view, animated mic button, iOS 26 Liquid Glass

## Architecture

```
Pod AI/
├── Models/
│   ├── Episode.swift      # Episode data (title, description, audioURL, transcript)
│   ├── Podcast.swift      # Podcast metadata
│   └── Category.swift     # Browse categories (unused)
├── Services/
│   ├── AudioPlayerService.swift      # AVPlayer-based playback, lock screen controls
│   ├── PodcastSearchService.swift    # iTunes Search API for podcast discovery
│   ├── RSSFeedService.swift          # XML parsing for podcast feeds
│   ├── TranscriptService.swift       # Transcript orchestration (cache → YouTube → Whisper)
│   ├── YouTubeTranscriptService.swift # YouTube search + transcript fetching
│   ├── OpenAIRealtimeService.swift   # WebSocket voice chat with GPT-4o
│   └── SecretsManager.swift          # API key loading from Secrets.plist
└── Views/
    ├── ContentView.swift          # App entry point
    ├── BrowseView.swift           # Home page with search bar
    ├── SearchView.swift           # Podcast search UI
    ├── ShowPageView.swift         # Podcast page with episode list
    ├── EpisodeDetailView.swift    # Episode details and play button
    ├── NowPlayingView.swift       # Full-screen player with voice button
    ├── VoiceInteractionView.swift # Voice chat UI with animated rings
    ├── MiniPlayerView.swift       # Compact player overlay
    ├── EpisodeRowView.swift       # Episode list item component
    └── Components/
        ├── SearchBarView.swift    # Liquid Glass search bar
        ├── GlowyMicButton.swift   # Animated mic button for voice interaction
        ├── CategoryCardView.swift # Category card (unused)
        └── PodcastRowView.swift   # Search result row
```

## Tech Stack

- **SwiftUI** for UI (iOS 26 Liquid Glass)
- **AVFoundation** + **MediaPlayer** for audio playback
- **iTunes Search API** for podcast discovery
- **YouTube Data API** for video search + transcript fetching (via `swift-youtube-transcript` SPM package)
- **OpenAI Whisper API** for transcription fallback
- **OpenAI Realtime API** (WebSocket) for voice chat
- API keys stored in `Secrets.plist` (not committed)

## Transcript Flow

```
1. Check local cache
2. If episode has youtubeVideoId → fetch YouTube transcript
3. Else → search YouTube by episode title → fetch transcript
4. Fallback → download audio + transcribe via Whisper API
5. Cache result for future use
```

## Data Flow

```
iTunes Search → PodcastSearchService → User selects podcast
    → RSS Feed → RSSFeedService → Episode List → User selects episode
    → AudioPlayerService plays audio
    → TranscriptService fetches transcript (YouTube or Whisper)
    → User taps voice button → OpenAIRealtimeService connects
    → Voice input → AI response with transcript context
```

## Voice Interaction Architecture

```
┌─────────────────────────────────────────────┐
│         Single AVAudioEngine                │
│                                             │
│  Mic → Input Node → [Echo Cancellation] → API
│           ↓                                 │
│  setVoiceProcessingEnabled(true)            │
│                                             │
│  API → Player Node → Mixer → Speaker        │
└─────────────────────────────────────────────┘
```

**Key Features:**
- Single engine for both input/output enables iOS echo cancellation
- Server VAD (Voice Activity Detection) for natural conversation
- Function calling: `resume_podcast`, `skip_forward`, `skip_backward`
- Interruption: User speech stops AI audio immediately
- Transcript truncated to 8000 chars to prevent API issues

---
> Source: [allensu02/Pod-AI](https://github.com/allensu02/Pod-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->

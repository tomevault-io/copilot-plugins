## lyriclens

> LyricLens is an AI-powered application that turns audio tracks—including music, stories, ads, and documentaries—into synchronized, visually stylistic videos. It leverages Google's Gemini models for transcription, content analysis, and image generation, combined with a versatile rendering pipeline to produce high-quality MP4 exports.

# LyricLens - AI Audio Visualizer & Music Video Generator

LyricLens is an AI-powered application that turns audio tracks—including music, stories, ads, and documentaries—into synchronized, visually stylistic videos. It leverages Google's Gemini models for transcription, content analysis, and image generation, combined with a versatile rendering pipeline to produce high-quality MP4 exports.

## Project Overview

-   **Frontend:** React (Vite), TypeScript, Tailwind CSS, **Shadcn UI**, **Framer Motion**.
-   **Backend:** Node.js (Express), FFmpeg (Video Stitching), yt-dlp (YouTube Import).
-   **Mobile:** **Capacitor** (Android & iOS support).
-   **AI Engine:** Google Gemini (via `@google/genai`), **LangChain** (Orchestration).
-   **Video Providers:** Google Veo (via Gemini) & **DeAPI** (3rd party).

### Core Features
*   **Director Mode:** A sophisticated LangChain-based agent system (Analyzer & Storyboarder) that analyzes content themes, motifs, and concrete visual objects to generate cohesive, cinematically paced visual storyboards.
*   **Visual Persona Support:** AI adapts its behavior based on video purpose:
    *   **Brand Specialist** (Commercial): Product-focused, clean lighting, no metaphors
    *   **Visual Poet** (Music Video): Metaphor literalism - shows actual objects mentioned in lyrics
    *   **Historian** (Documentary): Realism and accuracy focused
    *   **Viral Creator** (Social Short): Scroll-stopping visual hooks
*   **Concrete Motif Extraction:** The "Candle Fix" - when lyrics mention physical objects (candle, door, rain), the AI is forced to show these objects literally, not as abstract interpretations.
*   **Universal Content Handling:** Support for music, stories, ads, and documentaries.
*   **Video Generation:** Create animated videos using **Google Veo** or **DeAPI** (img2video).
*   **Translation:** Built-in subtitle translation to reach global audiences.
*   **Intent-Based Prompts:** 13+ specialized visual styles including "Commercial/Ad", "Manga", "Tutorial", and "Cinematic".
*   **Style Enhancement:** Art medium-specific keywords (watercolor: "bleeding edges", "wet-on-wet technique"; oil: "impasto brushwork", "canvas weave").
*   **Subject Consistency:** "Global Subject" input ensures character/object consistency across generated scenes.
*   **Advanced Prompt Engineering:** Automated linting, refinement, conflict detection, and motion prompt generation.
*   **Cross-Platform:** Runs on Web (Desktop/Mobile) and Native Mobile (Android/iOS).

## Architecture

The project uses a **hybrid architecture** combining client-side interactivity with powerful server-side and cloud-based processing:

1.  **AI Orchestration (Director Service):**
    *   **Persona System:** `getSystemPersona(purpose)` returns purpose-specific AI behavior rules.
    *   **Analyzer Agent:** Analyzes lyrics/transcript to determine themes, recurring motifs, and **concrete motifs** (physical objects to visualize).
    *   **Storyboarder Agent:** Generates detailed visual prompts mapped to the analysis, enforcing motif literalism.
    *   Uses **Zod** schemas for strictly typed AI outputs.

2.  **Rendering Pipeline:**
    *   **Client-Side:** `TimelinePlayer` renders visualizations (Wave, Particles, Bars) and subtitles on HTML5 Canvas.
    *   **Export:**
        *   **Cloud Render:** Uploads frames to the Node.js server for FFmpeg stitching (best for quality/speed).
        *   **Browser Render:** Uses `ffmpeg-wasm` for client-side encoding (best for privacy).
    *   **Speed Optimizations:** 720p rendering with FFmpeg upscaling, 24 FPS (cinema standard), optimized JPEG encoding.

## Key Files & Services

### Services (`services/`)
*   **`directorService.ts`**: The brain of the operation. Uses LangChain to orchestrate the Analyzer and Storyboarder agents.
*   **`geminiService.ts`**: Core bridge to Google's Gemini API for transcription and generation.
*   **`deapiService.ts`**: Integration with DeAPI for alternative video generation models.
*   **`translationService.ts`**: Handles batch translation of subtitles.
*   **`promptService.ts`**: Logic for prompt linting, refinement, Jaccard similarity checks (anti-repetition), and motion prompt generation.
*   **`imageService.ts`**: Abstracts image generation requests and applies style modifiers.
*   **`ffmpegService.ts`**: Video export with optimized 720p rendering and 24 FPS encoding.
*   **`transcriptionService.ts`**: Audio-to-text transcription with word-level timing (supports music, speech, documentaries, ads).

### Components (`components/`)
*   **`ConfigurationWizard.tsx`**: Multi-step flow for setting up a new project (Style, Mode, Ratio, Subject).
*   **`TimelinePlayer.tsx`**: Advanced player with multiple visualizers (Particles, Wave, Circular) and synchronized subtitle display.
*   **`ImageGenerator.tsx`**: Motion-enhanced card for viewing/editing artwork.
*   **`TranscriptList.tsx`**: Interactive list with optimized rendering and smooth active state animations.

### Core Logic
*   **`hooks/useLyricLens.ts`**: The main state machine. Manages the lifecycle of File Load -> Transcribe -> Analyze -> Generate -> Export.

## Setup & Usage

### Prerequisites
*   Node.js (v18+)
*   FFmpeg installed in system PATH (required for Server-side rendering).
*   Google Gemini API Key.
*   (Optional) DeAPI Key for alternative video generation.
*   (Optional) Android Studio for mobile builds.

### Environment
Create a `.env.local` file in the root:
```env
GEMINI_API_KEY=your_gemini_key
VITE_DEAPI_API_KEY=your_deapi_key_optional
```

### Commands

**Development**
| Command | Description |
| :--- | :--- |
| `npm run dev:all` | **Recommended.** Runs Vite frontend + Express server. |
| `npm run dev` | Runs only the frontend. |
| `npm run server` | Runs only the backend server. |

**Testing**
| Command | Description |
| :--- | :--- |
| `npm test` | Runs the test suite (`test/runTest.ts`). |

**Mobile (Capacitor)**
| Command | Description |
| :--- | :--- |
| `npm run cap:sync` | Syncs web assets to native projects. |
| `npm run cap:android` | Opens Android Studio. |
| `npm run cap:ios` | Opens Xcode (macOS only). |
| `npm run build:mobile` | Builds frontend and syncs to mobile projects. |

## Development Conventions

*   **Agents:** Use `directorService` for complex AI tasks requiring multi-step reasoning.
*   **Validation:** Use `zod` schemas for all AI JSON outputs.
*   **State:** Business logic resides in `useLyricLens.ts`; components should be presentational.
*   **Styling:** Shadcn UI + Tailwind CSS. Use `framer-motion` for transitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lordpython)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lordpython)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

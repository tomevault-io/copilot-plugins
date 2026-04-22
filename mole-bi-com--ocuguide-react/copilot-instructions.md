## ocuguide-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OcuGuide is a React-based web application that provides customized information for cataract surgery patients in Korean. It features patient information management, step-by-step surgery guidance with audio support, an AI-powered chatbot for consultations, and usage statistics tracking.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm build

# Run tests
npm test
```

## Environment Setup

Create a `.env` file in the project root with the following variables:

```
REACT_APP_SUPABASE_URL=your_supabase_url
REACT_APP_SUPABASE_KEY=your_supabase_key
REACT_APP_OPENAI_API_KEY=your_openai_api_key
```

## Architecture

### Frontend Structure

- **React Router**: Application uses React Router 6 for navigation with protected routes
- **State Management**: Uses React Context API via multiple providers:
  - `AuthProvider`: Handles authentication state and user sessions
  - `PatientProvider`: Manages patient data throughout the application
  - `AppProvider`: Manages application-wide state and settings

### External Services

- **Supabase**: Used for data storage and authentication
- **OpenAI APIs**:
  - GPT-4o-mini for chat responses and patient diagnosis
  - TTS-1 for Text-to-Speech functionality
  - Whisper for Speech-to-Text features

### Key Feature Implementation

1. **Audio Synchronization**:
   - Audio files are synced with text using the `useAudio` hook
   - Recent improvements include karaoke-style text highlighting during playback

2. **Voice Interactions**:
   - Audio recording implemented in `AudioRecorder.jsx`
   - Speech recognition via Web Speech API in the `useSpeechRecognition` hook

3. **Chatbot**:
   - Implemented in `ChatInterface.jsx` using OpenAI API
   - Recent updates focus on voice input rather than text input

4. **Surgery Information Flow**:
   - Multi-step information presentation with progress tracking
   - Audio playback controls to ensure patients listen before proceeding

## Code Organization

- `/src/components/`: UI components organized by feature
- `/src/context/`: Context providers for state management
- `/src/hooks/`: Custom hooks for audio, localStorage, and speech recognition
- `/src/pages/`: Page components that utilize the Layout component
- `/src/services/`: API integrations (Supabase, OpenAI) and utility services

## Recent Development Focus

Based on the recent commits and plan.md:

1. Improving audio-text synchronization with karaoke-style highlighting
2. Adding features to ensure patients listen to audio before proceeding
3. Restructuring patient information and post-surgery sections
4. Enhancing the chatbot's voice interaction capabilities

## Important Files

- `src/App.js`: Entry point with route definitions and context providers
- `src/services/api.js`: Core API functionality for both Supabase and OpenAI
- `src/hooks/useAudio.js`: Audio playback management
- `src/components/SurgeryInfo/InfoStep.jsx`: Handles step-by-step surgery information
- `src/components/Chatbot/ChatInterface.jsx`: Manages chatbot interactions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mole-bi-com) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

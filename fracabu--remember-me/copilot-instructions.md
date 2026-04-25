## remember-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Install dependencies**: `npm install`
- **Start development server**: `npm run dev`
- **Build for production**: `npm run build`
- **Preview production build**: `npm run preview`

## Styling Setup

The project uses **Tailwind CSS** with PostCSS:
- Configuration in `tailwind.config.js` with custom color palette and dark mode support
- Main CSS file at `src/index.css` with Tailwind directives and custom components
- PostCSS configuration in `postcss.config.js`
- **Important**: Uses proper Tailwind installation, not CDN (for production compatibility)

## Project Setup

This is a React + TypeScript + Vite application that requires a Gemini API key to function. The API key can be configured via environment variables or runtime UI:

- **Environment API Key**: Set `VITE_GEMINI_API_KEY` in `.env.local` (recommended for development)
- **Runtime Configuration**: If no environment key exists, the app shows a sidebar for API key input
- **API Key Security**: Keys are never stored persistently - only kept in memory during the session

## Architecture Overview

### Core Application Flow
Voice-to-reminder converter using Google's Gemini AI:

1. **Voice Recording**: Web Speech API captures and transcribes speech in browser
2. **AI Processing**: Gemini AI extracts structured data (title, description, date, time) via `processVoiceInput()`
3. **Storage**: Reminders persist in localStorage under `'remember-me-reminders'` key
4. **State Management**: React hooks with no external state management library

### Key Components & Services

- **src/App.tsx**: Main container with global state (reminders, API key, processing status, error handling). Handles API key validation and adaptive UI (shows sidebar if no environment key)
- **src/components/Recorder.tsx**: Web Speech API integration with TypeScript definitions for speech recognition interfaces
- **src/components/SettingsModal.tsx & SettingsSidebar.tsx**: Dual API key configuration - modal for environment keys, sidebar for runtime input
- **src/components/ReminderList.tsx & ReminderCard.tsx**: Reminder display with time-based categorization, calendar export, re-recording, and AI features
- **src/services/geminiService.ts**: Three main AI functions with structured schema:
  - `processVoiceInput()`: Speech → structured reminders
  - `generateTaskSuggestions()`: Task-specific productivity advice
  - `sendChatMessage()`: Conversational task assistance
- **src/utils/timeUtils.ts**: Time categorization logic (Oggi/Domani/Scaduto/etc.) with Italian localization
- **src/contexts/ThemeContext.tsx**: Dark/light theme system with localStorage persistence and system preference detection

### Advanced Features

- **Re-recording Mode**: Updates existing reminders by integrating new voice input with existing content rather than replacing
- **AI Chat Integration**: Per-reminder conversational assistance for planning and troubleshooting
- **Smart Time Parsing**: Italian locale support with AM/PM inference, relative date calculation, and 24-hour format handling
- **Theme System**: Dark/light modes with Tailwind CSS classes and automatic persistence
- **Time Categorization**: Reminders grouped by urgency (urgent/soon/upcoming/past) with color coding

### Technical Configuration

- **Vite Config**: Standard Vite configuration with path aliases and PostCSS integration
- **Tailwind Integration**: Proper Tailwind CSS installation with PostCSS, custom color palette with primary/secondary/accent colors and dark mode support
- **ESM Imports**: Uses import maps for React 19 and Google GenAI from esm.sh CDN  
- **TypeScript**: Comprehensive type definitions in `src/types/index.ts` with time-enhanced reminder interfaces
- **CSS Architecture**: Custom component classes in `src/index.css` with Tailwind utilities

### AI Processing Details

- **Model**: Uses `gemini-2.5-flash` for fast processing
- **Schema Validation**: Structured JSON responses with required fields (title, description, date, time)
- **Italian Locale**: Time and date formatting using `it-IT` locale with timezone awareness
- **Context Integration**: Chat and suggestions maintain context about specific reminders and user timezone
- **Error Handling**: Comprehensive error states for API failures, empty responses, and JSON parsing issues

The application is client-side only with no backend server required.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fracabu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

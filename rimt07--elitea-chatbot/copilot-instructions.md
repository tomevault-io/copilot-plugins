## elitea-chatbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React TypeScript chatbot application that integrates with the Elitea AI API. The application provides a floating button interface for AI-powered conversations with real-time streaming responses, participant management, and conversation handling.

## Development Commands

- `npm run dev` - Start development server (Vite)
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## Architecture

### Core Components Structure
- `src/components/ChatBot.tsx` - Main chatbot interface component
- `src/components/FloatingButton.tsx` - Entry point floating button UI
- `src/components/ParticipantPanel.tsx` - Manages AI participants and their settings
- `src/components/ConversationPanel.tsx` - Handles conversation selection/creation
- `src/components/MessagePanel.tsx` - Chat message display and input

### API Integration
- `src/services/eliteaApi.ts` - Main API service class with methods for:
  - Creating conversations (`/chat/conversations/prompt_lib/42`)
  - Managing participants (`/chat/participants/prompt_lib/42/{conversationId}`)
  - Sending messages with streaming support (`/prompt_lib/predict/prompt_lib/42`)
  - Fetching available participants (`/applications/applications/prompt_lib/42`)

### Configuration & Types
- `src/config/api.ts` - Environment-based API configuration with validation
- `src/types/index.ts` - TypeScript interfaces for API responses, conversations, participants, and messages

### Environment Variables Required
```
VITE_ELITEA_API_URL=https://nexus.elitea.ai/api/v1
VITE_ELITEA_BEARER_TOKEN=your_bearer_token_here
VITE_ELITEA_COOKIE=your_cookie_here
```

## Key Implementation Details

- Uses streaming API responses for real-time chat experience
- Handles both streaming (`text/event-stream`) and JSON response formats
- Participant system allows multiple AI models per conversation with individual settings (temperature, max_tokens, etc.)
- Error handling includes proper API response validation and user-friendly error messages
- Security: Bearer tokens and cookies are handled via environment variables only

## Tech Stack

- React 18 + TypeScript
- Vite for development/build
- Tailwind CSS for styling
- Lucide React for icons
- ESLint for code quality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rimt07)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rimt07)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## writing-helper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Development Server
```bash
yarn dev          # Start development server with Turbopack (uses Yarn 4.6.0)
```

### Build & Production
```bash
yarn build        # Build for production
yarn start        # Start production server
```

### Code Quality
```bash
yarn lint         # Run ESLint
```

## Core Architecture

This is a Next.js 15 AI writing assistant with TypeScript and Tailwind CSS. The application helps users generate content using various LLM APIs with streaming support and enhanced security.

### Key Components Structure

- **API Proxy Layer**: 
  - `src/app/api/proxy/route.ts`: Traditional proxy with CORS handling and 10-minute timeout
  - `src/app/api/stream-proxy/route.ts`: Streaming proxy for real-time content generation with Server-Sent Events
- **Core Types** (`src/app/lib/types.ts`): Defines `PromptStyle`, `WritingRequest`, `PolishRequest` interfaces with 8-dimensional style system
- **API Client** (`src/app/lib/api.ts`): Dual-mode API client supporting both traditional and streaming content generation
- **Security Layer** (`src/app/lib/secureApiKey.ts`): Client-side API key encryption and management system
- **UI Components**:
  - `StreamingContent.tsx`: Real-time content display with typewriter effect
  - `ApiSettings.tsx`: Enhanced API configuration with security features
  - `WritingAssistant.tsx`: Main interface supporting both streaming and traditional modes

### LLM Provider Support

The application handles multiple LLM APIs through unified proxies with streaming support:
- **OpenAI**: GPT-4 series with streaming chat completions
- **Grok (xAI)**: `grok-3-latest` model with streaming support
- **Ollama**: Local models with `/api/generate` endpoint and streaming
- **DeepSeek**: OpenAI-compatible with DeepSeek models and streaming

### Key Features

1. **Streaming Content Generation**: Real-time content display with typewriter effect and visual progress indicators
2. **Writing Assistant**: 8-dimensional style system (language, structure, narrative, emotion, thinking, uniqueness, cultural, rhythm)
3. **AI Text Optimizer**: Removes AI characteristics with statistical feature optimization
4. **WeChat Formatter**: Specialized formatting for WeChat publishing
5. **Article Polisher**: Content refinement with different polish modes

## Important Development Notes

### Streaming Architecture
- `/api/stream-proxy` handles Server-Sent Events for real-time content generation
- `StreamingContent` component provides typewriter effect with 20ms/character timing
- Both streaming and traditional modes available with user toggle

### API Security Model
- **User-managed API keys**: Users input their own API keys through secure UI
- **Client-side encryption**: XOR encryption with browser fingerprint-based keys
- **Automatic expiration**: 24-hour default, 7-day "remember me" option
- **Secure CORS**: Origin-based validation via `ALLOWED_ORIGINS` environment variable

### API Request Flow
All LLM requests go through proxy endpoints that:
- Auto-detect provider type from URL
- Handle different request/response formats
- Provide comprehensive error handling with user-friendly messages
- Support both streaming and traditional response modes

### Content Generation System
The 8-dimensional style system includes:
- **Language**: Sentence patterns, word choice (formality 1-5), rhetoric techniques
- **Structure**: Paragraph length, transitions, hierarchy patterns  
- **Narrative**: Perspective, time sequence, narrator attitude
- **Emotion**: Intensity (1-5), expression style, tone
- **Thinking**: Logic patterns, depth (1-5), rhythm
- **Uniqueness**: Signature phrases, imagery systems
- **Cultural**: Allusions, knowledge domains
- **Rhythm**: Syllable patterns, pause patterns, tempo

## Configuration Files

- `next.config.mjs`: Next.js configuration with Turbopack
- `tailwind.config.js`: Tailwind CSS configuration with custom styling
- `eslint.config.mjs`: ESLint configuration with Next.js rules
- `tsconfig.json`: TypeScript configuration with strict type checking
- `.env.local`: Environment variables (ALLOWED_ORIGINS for CORS security)
- `.cursor/rules/next.mdc`: Cursor editor rules for Next.js development

## Security Implementation

### API Key Management
- Keys stored with XOR encryption + Base64 encoding
- Browser fingerprint-based encryption keys
- No server-side key storage - purely client-side
- Automatic cleanup on expiration

### CORS Protection
- Environment-variable controlled origin validation
- No wildcard CORS in production
- Request origin verification on all API endpoints

### Content Security
- Input validation on all user inputs
- Secure proxy handling with timeout controls
- Error messages sanitized to prevent information leakage

## Package Manager
Uses Yarn 4.6.0 as specified in package.json packageManager field. All commands should use `yarn` instead of `npm`.

---
> Source: [GeekyWizKid/writing-helper](https://github.com/GeekyWizKid/writing-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

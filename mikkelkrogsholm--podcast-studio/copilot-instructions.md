## podcast-studio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a browser-based podcast studio built on OpenAI's Realtime API, designed for conducting live two-way conversations between a human host (Mikkel) and an AI co-host (Freja). The system records two separate audio tracks for post-production flexibility.

## Architecture

### Monorepo Structure
- **apps/web**: Next.js/React frontend for UI and audio capture (port 4200)
- **apps/api**: Node.js/TypeScript backend for OpenAI proxy and data persistence (port 4201)
- **packages/shared**: Shared TypeScript types and Zod utilities

### Core Technologies
- **Frontend**: Next.js, React, TypeScript (strict mode)
- **Backend**: Node/Express or Next API routes, TypeScript
- **Database**: SQLite with Drizzle ORM
- **Audio**: WebRTC for low-latency streaming, MediaRecorder API for dual-track recording
- **API**: OpenAI Realtime API with WebRTC/WebSocket connections

### Data Flow
1. Frontend captures local microphone (Mikkel track) via MediaDevices API
2. WebRTC connection established to OpenAI Realtime API using ephemeral tokens
3. AI audio output (Freja track) captured separately via MediaStreamDestination
4. Both tracks uploaded in chunks to backend for persistence
5. Live transcripts displayed and saved to database

## Development Commands

```bash
# Install dependencies
pnpm install

# Development servers (run in parallel)
pnpm dev:api    # Start API server on port 4201
pnpm dev:web    # Start web frontend on port 4200

# Testing
pnpm test       # Run all tests
pnpm test:api   # Run API tests only  
pnpm test:web   # Run web tests only
pnpm test:e2e   # Run Playwright E2E tests

# Code quality
pnpm lint       # Run ESLint
pnpm typecheck  # Run TypeScript type checking
pnpm format     # Run Prettier

# Pre-push validation (run before pushing to avoid CI failures)
pnpm typecheck && pnpm test && pnpm lint

# Database
pnpm db:migrate # Run database migrations
pnpm db:seed    # Seed development data
```

## Implementation Steps (TDD Approach)

The project follows a 12-step incremental build plan (see context/steps/):

1. **Step 0**: Repository skeleton with basic tooling
2. **Step 1**: OpenAI connection and token generation
3. **Step 2**: WebRTC/WebSocket handshake
4. **Step 3**: Local microphone capture and upload
5. **Step 4**: AI audio output as separate track
6. **Step 5**: Auto-save and crash recovery
7. **Step 6**: Live transcript display and persistence
8. **Step 7**: Playground-style controls (model, voice, temperature)
9. **Step 8**: Persona and context prompts
10. **Step 9**: File download and export
11. **Step 10**: Session management (stop/retry/interrupt)
12. **Step 11**: Session history and details view
13. **Step 12**: Extension hooks for future features

## Key Implementation Details

### Audio Recording
- Two separate WAV tracks: mikkel.wav and freja.wav
- Format: PCM16, 48kHz, mono
- Chunks uploaded every 1-2 seconds for auto-save
- Files stored at: `sessions/{id}/{speaker}.wav`

### OpenAI Realtime Integration
- Backend mints short-lived session tokens (never expose API key to client)
- WebRTC preferred for lowest latency
- Server-side VAD with ~900ms default silence threshold
- Maximum session length: 30 minutes (OpenAI limit)

### Database Schema
- **sessions**: Stores session metadata, settings, and prompts
- **messages**: Transcript events with speaker, text, timestamp
- **audio_files**: References to audio file paths and metadata

### Testing Strategy
- TDD for API endpoints and database operations
- Manual testing for WebRTC/audio features requiring browser permissions
- Live tests (tagged @live) only run when OPENAI_API_KEY is set
- **Important**: Vitest runs `.test.ts` files, Playwright runs `.spec.ts` files (keep them separate)
- **CI Tip**: Always run `pnpm typecheck && pnpm test && pnpm lint` before pushing

## Agent-Specific Guidance

### audio-engineer Agent
Focus on frontend audio/WebRTC implementation. Key responsibilities:
- Set up dual-track recording (local mic + remote AI)
- Ensure low latency WebRTC connection
- Implement testable audio capture with fake-mic flag support

### test-writer Agent  
Write minimal tests first following TDD principles:
- Extract acceptance criteria from step files
- Create Vitest/Playwright test skeletons
- Run tests expecting failure, then guide implementation to green

### backend-surgeon Agent
Build precise API endpoints with validation:
- One route at a time following TDD
- Use Zod for request/response validation
- Clear 400/200 response scenarios
- No premature abstraction

## Security Considerations
- API keys only in backend environment variables
- Ephemeral tokens for client-side OpenAI connections
- CORS locked to application origin in production
- Structured logging without sensitive data

## Documentation

Additional documentation is available in the `docs/` folder:

### Core Technical Guides
- **[testing-guide.md](docs/testing-guide.md)** - Complete guide for running API and E2E tests, including troubleshooting
- **[testing-gotchas.md](docs/testing-gotchas.md)** - Common test failures and solutions for CI/CD issues
  - React Context mocking with Vitest
  - TypeScript strict checks in tests
  - Mock persistence for nested components
- **[webrtc-handshake.md](docs/webrtc-handshake.md)** - WebRTC connection implementation details and security model
- **[openai-integration.md](docs/openai-integration.md)** - OpenAI Realtime API setup and connection guide
- **[branch-summary.md](docs/branch-summary.md)** - Feature branch implementation summary and progress tracking

### UI/UX and Design
- **[nordic-ui-topbar.md](docs/nordic-ui-topbar.md)** - Nordic-inspired UI design with clean TopBar component
  - Minimalist design principles with focus on functionality
  - TopBar component architecture and state management
  - Bilingual UI elements and controls
- **[internationalization.md](docs/internationalization.md)** - i18n implementation guide for Danish/English support
  - Danish and English language support
  - Translation management and locale switching
  - UI text requirements for bilingual interface

### Implementation Steps
- **[step-03-implementation.md](docs/step-03-implementation.md)** - Technical details for Step 3 audio recording implementation
- **[step-04-dual-track-recording.md](docs/step-04-dual-track-recording.md)** - Dual-track voice recording with OpenAI Realtime API
- **[step-05-auto-save-recovery.md](docs/step-05-auto-save-recovery.md)** - Auto-save functionality and crash recovery mechanisms
- **[step-06-transcript-persistence.md](docs/step-06-transcript-persistence.md)** - Live transcript display and database persistence
- **[step-07-playground-controls.md](docs/step-07-playground-controls.md)** - Playground-style controls for model, voice, and temperature settings
- **[step-08-persona-context.md](docs/step-08-persona-context.md)** - Persona and context prompts for AI customization and conversation topics
- **[step-09-file-download-export.md](docs/step-09-file-download-export.md)** - File download and export functionality
  - Audio file download endpoints
  - Transcript export in text and JSON formats
  - Combined download with session metadata
- **[step-10-session-management.md](docs/step-10-session-management.md)** - Session management with stop/interrupt functionality
  - Session finish endpoint with duration calculation
  - Stop and interrupt button implementation
  - Reconnect logic with exponential backoff
- **[step-11-session-history.md](docs/step-11-session-history.md)** - Session history and details pages
  - GET /api/sessions with pagination and sorting
  - Session list page with responsive table/card view
  - Session details page with complete metadata and download links
  - Bilingual support for all UI elements
- **[step-12-event-hooks.md](docs/step-12-event-hooks.md)** - Event emitter and extension hooks system
  - Minimal EventEmitter implementation for future features
  - session:completed event type with no-op subscriber
  - Foundation for show notes, RSS feeds, analytics
- **[step-12-implementation-summary.md](docs/step-12-implementation-summary.md)** - Complete Step 12 implementation details
  - TDD process and test coverage
  - Code changes with line numbers
  - Future extension points

### UI Components and Styling
The application uses a modern, minimalist Nordic-inspired design with:
- **Tailwind CSS** for utility-first styling
- **Next.js App Router** for routing and layouts
- **React Components** with TypeScript for type safety
- **Nordic Design Principles**: Clean, functional, accessible
- **TopBar Component**: Centralized control center for recording and settings
- **Dark mode support** via Tailwind's dark: prefix
- **Responsive design** for desktop and mobile layouts
- **Accessibility-first** approach with proper ARIA labels and keyboard navigation
- **Bilingual Interface**: All UI elements support Danish and English

## Important Constraints
- **No over-engineering**: Build only what's needed for the current step
- **Persona/context locked per session**: Cannot be changed mid-recording
- **Auto-save everything**: Every chunk and event persisted immediately
- **Raw data first**: Store unprocessed audio and transcripts
- Always make sure the UI is danish and english
- We are in september of 2025

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikkelkrogsholm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

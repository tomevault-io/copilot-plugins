## study-timer-for-kandel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js-based study timer application implementing the **3-Pass Method** for focused learning. The app guides users through 5 sequential phases (MAPPING, IMAGE HACK, SCAN, RECALL, REST) with timed intervals, visual progress tracking, and audio notifications.

## Development Commands

```bash
# Start development server
npm run dev         # Opens at http://localhost:3000

# Production build
npm run build       # Creates optimized production build
npm start           # Serves production build locally

# Deployment (Vercel)
vercel              # Preview deployment
vercel --prod       # Production deployment
```

**Node.js requirement:** >=18.18.0 (specified in package.json engines)

## Architecture

### Core Application Flow

1. **Entry Point**: [pages/index.jsx](pages/index.jsx) - Single-page app entry with SEO metadata
2. **Main Component**: [components/StudyTimer.jsx](components/StudyTimer.jsx) - All timer logic and UI
3. **Global Styles**: [styles/globals.css](styles/globals.css) - Inter font import and dark theme base

### Phase System Architecture

The timer is built around a **phases array** ([components/StudyTimer.jsx:3-51](components/StudyTimer.jsx#L3-L51)) containing 5 sequential study phases. Each phase object defines:
- `id`, `name`, `caption`, `detail` - Display text and metadata
- `duration` - Phase length in seconds
- `color` - Theme color for UI accents
- `number` - Display identifier
- `important` - Boolean flag for critical phases (IMAGE HACK, RECALL)

### State Management Pattern

Uses React hooks with no external state library:
- `currentPhaseIndex` - Tracks active phase (0-4)
- `timeRemaining` - Countdown in seconds for current phase
- `isRunning` - Play/pause state
- `sessionCount` - Tracks completed full cycles

**Auto-advance behavior** ([components/StudyTimer.jsx:82-92](components/StudyTimer.jsx#L82-L92)): When a phase timer hits 0, it plays a notification sound and automatically advances to the next phase, or cycles back to phase 0 and increments sessionCount.

### Progress Calculation

Two types of progress are computed in real-time:
1. **Current phase progress** ([components/StudyTimer.jsx:132](components/StudyTimer.jsx#L132)) - Percentage through active phase
2. **Total progress** ([components/StudyTimer.jsx:71-74](components/StudyTimer.jsx#L71-L74)) - Elapsed time across all phases relative to total duration

The total progress bar ([components/StudyTimer.jsx:456-476](components/StudyTimer.jsx#L456-L476)) is segmented with each segment's flex basis proportional to its phase duration, creating a visually weighted progress indicator.

### Audio Notification System

Uses **Web Audio API** ([components/StudyTimer.jsx:97-109](components/StudyTimer.jsx#L97-L109)) to synthesize a 440Hz sine wave notification tone when phases complete. This approach avoids file dependencies and works across platforms without requiring audio file assets.

### UI Layout Structure

Two-column grid layout ([components/StudyTimer.jsx:136-153](components/StudyTimer.jsx#L136-L153)):
- **Left column** - Main timer display with large time countdown, phase details, and playback controls
- **Right column** - Queue/playlist sidebar showing all phases with status indicators (completed checkmark, active animation, upcoming number)

All styling is inline CSS-in-JS (no external CSS modules or styled-components). The design uses a dark theme (#09090b background) with phase-specific accent colors.

## Key Implementation Details

**Timer precision**: Uses setInterval with 1-second ticks ([components/StudyTimer.jsx:79-81](components/StudyTimer.jsx#L79-L81))

**Phase navigation**: Users can click any phase in the queue to jump to it, or use prev/next buttons (disabled at boundaries)

**Session tracking**: Completing all 5 phases increments sessionCount and resets to phase 0

**Responsive considerations**: Fixed grid layout optimized for desktop (maxWidth: 1100px). Mobile responsiveness would require media queries.

## No External Configuration

This project has no TypeScript, ESLint config, testing framework, or build configuration beyond Next.js defaults. It uses plain JavaScript (.jsx/.js files) throughout.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gui692) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

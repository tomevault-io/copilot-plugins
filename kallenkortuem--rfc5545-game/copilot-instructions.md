## rfc5545-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15.4.2 gamified learning application for RFC 5545 (iCalendar specification). It uses:
- React 19.1.0 with TypeScript
- App Router architecture
- Tailwind CSS v4 with PostCSS
- Turbopack for development
- Local storage for progress persistence

## Essential Commands

```bash
# Development
npm run dev        # Start development server with Turbopack on http://localhost:3000

# Production
npm run build      # Create optimized production build
npm run start      # Run production server

# Code Quality
npm run lint       # Run ESLint checks
```

## Architecture

### Core Systems
- `src/lib/gameData.ts` - Game progression structure, chapters, levels, achievements
- `src/lib/gameState.ts` - React Context for game state management with localStorage
- `src/lib/icalParser.ts` - RFC 5545 compliant parser and validator
- `src/lib/challenges.ts` - Challenge definitions and validation rules

### Page Structure
- `src/app/` - App Router pages
- `src/app/play/[chapterId]/[levelId]/` - Dynamic route for challenges
- `src/components/` - React components

### Key Components
- `GameDashboard` - Main game interface with chapter progression
- `ChallengeInterface` - Code editor and validation for challenges
- `CodeEditor` - Custom iCalendar editor with syntax highlighting
- `ValidationPanel` - Real-time RFC 5545 validation feedback
- `ReferencePanel` - Searchable RFC 5545 quick reference

## Game Features

- **4 Main Chapters**: Basics, Data Types, Components, Recurrence
- **Level Types**: Tutorial, Practice, Challenge, Boss
- **Progression System**: XP, achievements, star ratings
- **Real-time Validation**: Instant feedback on iCalendar code
- **Hint System**: Progressive hints that affect scoring

## Development Notes

- Game state persists in localStorage with key 'rfc5545-game-progress'
- Welcome screen shown once with key 'rfc5545-welcome-seen'
- TypeScript path alias: `@/*` maps to `./src/*`
- Dark theme only with slate color palette
- Uses Geist font family (Sans and Mono variants)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kallenkortuem)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kallenkortuem)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

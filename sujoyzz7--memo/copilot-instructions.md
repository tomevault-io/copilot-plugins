## memo

> This file contains guidelines and commands for agentic coding agents working on the Memio memory training application.

# Memio - Agent Development Guide

This file contains guidelines and commands for agentic coding agents working on the Memio memory training application.

## Build & Development Commands

- No need to run any commands for now
- Ask for help if you need it
- No testing required other than type check with `pnpm tsc`

## Project Architecture

### Tech Stack

- **Framework**: Next.js 16.1.1 with App Router
- **Language**: TypeScript 5.x (strict mode disabled)
- **UI**: shadcn/ui with New York style theme
- **Styling**: Tailwind CSS v4 with custom color system
- **State Management**: Zustand for local state
- **Icons**: Lucide React
- **Charts**: Recharts for statistics
- **Animations**: Anime.js

### Project Structure

```
src/
├── app/                    # Next.js App Router pages
├── components/             # Reusable UI components
├── features/               # Feature-specific modules
│   ├── core/              # Shared utilities and types
│   ├── numbers/           # Number memory game
│   ├── binary/            # Binary memory game
│   ├── cards/             # Card matching game
│   ├── chessboard/        # Chessboard memory game
│   └── stats/             # Statistics and charts
├── utils/                  # Utility functions
└── data/                   # Static data files
```

## Code Style Guidelines

### TypeScript Guidelines

- Use comprehensive TypeScript interfaces
- Shared types in `src/features/core/types/`
- Game-specific type definitions in feature directories
- Session record typing with timestamp support

### Styling Approach

- Use Tailwind CSS v4 with custom color system
- CSS custom properties for theming
- Consistent spacing and typography scale
- Dark mode support with custom variants

## State Management

### Zustand Stores

- Use Zustand for local state management
- Feature-specific stores in `src/features/*/hooks/`
- Consistent store patterns across features
- Storage abstraction for persistence

### Storage System

- Custom `Storage<T>` class wrapping localStorage
- Timestamp support for session records
- Session records with accuracy tracking
- Configuration persistence

## Development Workflow

### Feature Development

1. Create feature directory with proper structure
2. Implement types and interfaces first
3. Build UI components with proper typing
4. Add state management with Zustand
5. Implement storage and persistence
6. Add validation and error handling
7. Test thoroughly before completion

### Game Architecture

- Consistent game architecture across different types
- Shared utilities for storage and configuration
- Timer-based gameplay mechanics
- Progress tracking and statistics

### Control Panel Pattern

- Reusable control panel wrapper
- Consistent layout for game controls and status
- Timer integration for game mechanics
- Action buttons with proper feedback

---
> Source: [Sujoyzz7/memo](https://github.com/Sujoyzz7/memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->

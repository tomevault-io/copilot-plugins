## passant-prototype

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React Native notes application built with Expo, featuring location mentions, animated suggestion pills, and a beautiful dark/light theme system. The app is configured specifically for iOS development.

## Commands

### Development

- `npm start` or `npm run ios` - Start Expo development server for iOS
- `npm run lint` - Run ESLint to check for code issues
- `npm run lint:fix` - Automatically fix ESLint errors
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check if code formatting is correct
- `npm run typecheck` - Run TypeScript type checking
- `npm run check-all` - Run all quality checks
- `npm run setup-hooks` - Set up git pre-commit hooks

### Installation

- `npm install` - Install all dependencies

## Architecture

### Core Stack

- **React Native** with Expo 53.0.22
- **TypeScript** with partial strict mode (noImplicitAny: false)
- **React Navigation** for navigation (bottom tabs + native stack)
- **AsyncStorage** for data persistence

### Project Structure

```
src/
├── components/     # Reusable UI components (MentionPill, RichTextRenderer, SuggestionPills)
├── context/        # React contexts (NotesContext for state, ThemeContext for theming)
├── screens/        # Screen components for each route
├── navigation/     # Navigation setup (AppNavigator with tabs and stack)
├── constants/      # App constants and configuration values
├── config/         # App configuration
└── types/          # TypeScript type definitions
```

### Key Architectural Patterns

1. **Context-based State Management**:
   - `NotesContext` manages all notes data and operations
   - `ThemeContext` manages theme switching and provides theme colors

2. **Navigation Structure**:
   - Bottom tab navigator with 4 main screens (Activity, My Lists, Explore, Profile)
   - Stack navigator for modal screens (NoteEditor)
   - All navigation wrapped in NavigationContainer with dynamic theme

3. **Component Patterns**:
   - Rich text editing using `@10play/tentap-editor`
   - Location mentions with react-native-controlled-mentions
   - Animated UI components using react-native-reanimated
   - Haptic feedback for iOS interactions

4. **Theme System**:
   - Dynamic light/dark theme switching
   - Theme colors provided via context
   - Automatic status bar styling based on theme

### Key Features Implementation

- **Suggestion Pills**: Interactive templates above keyboard for quick note creation
- **Location Mentions**: Type `@` to trigger full-width typeahead that slides up from keyboard
  - FullWidthTypeahead component with addresses and smooth animations
  - Keyboard-aware positioning with automatic editor scrolling
  - Real-time filtering of suggestions as user types
- **Smart Formatting**: Auto-conversion of markdown-style bullets and continuation
- **Date Organization**: Notes automatically grouped by date (Today, This Month, etc.)

## Development Guidelines

- The app is iOS-only with SF Symbols and iOS-specific haptic feedback
- All code should follow the existing ESLint and Prettier configuration
- TypeScript is configured with relaxed rules (no implicit any allowed)
- Components should use the existing theme system for colors
- New features should integrate with existing context providers
- Pre-commit hooks enforce code quality (run `npm run setup-hooks` after cloning)
- GitHub Actions CI/CD runs on all pushes and PRs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pdugan20)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pdugan20)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

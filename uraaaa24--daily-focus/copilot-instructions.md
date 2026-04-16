## daily-focus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Start development server**: `npm start` or `npx expo start`
- **Platform-specific development**: 
  - `npm run android` - Start on Android emulator
  - `npm run ios` - Start on iOS simulator  
  - `npm run web` - Start web version
- **Code quality**: `npm run lint` - Run ESLint with Expo config

## Architecture Overview

This is a React Native/Expo productivity timer app using file-based routing with the following structure:

### Core Technologies
- **Expo SDK ~53.0** with new architecture enabled
- **React Native 0.79.5** with React 19.0
- **Expo Router** for file-based navigation with typed routes
- **MMKV** for local data persistence
- **TypeScript** with strict mode enabled

### App Structure
- **`app/_layout.tsx`**: Tab navigation layout with Home and History tabs
- **`app/index.tsx`**: Main timer interface with start/stop functionality and session tracking
- **`app/history.tsx`**: Session history display using FlatList

### Data Flow
- Timer state managed with React hooks (`useState`, `useEffect`)
- Session persistence handled via `react-native-mmkv` with `useMMKVString` hook
- History stored as JSON array in MMKV storage under 'history' key
- Sessions are automatically saved when timer is stopped

### Key Patterns
- Uses absolute imports with `@/*` path mapping
- Follows Expo/React Native styling with StyleSheet
- Timer uses `setInterval` with proper cleanup in `useEffect`
- Data persistence is handled synchronously with MMKV

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uraaaa24) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

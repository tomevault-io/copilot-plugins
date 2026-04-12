## tsuriai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tsuriai (釣り合い) is a two-sided fish marketplace app connecting fishermen/auctions with international buyers. Built with React Native, Expo SDK 54, and the Ignite boilerplate (v11.3.2). Uses bun as the package manager.

### App Concept
- **Sellers**: Individual fishermen, fishing boats/crews, auction houses
- **Buyers**: Restaurants/chefs, wholesalers, retailers, consumers
- **Transactions**: Live auctions (セリ), buy-now, pre-orders/contracts
- **Design**: Japanese fish market aesthetic (chirashi style, Tsukiji-inspired)

## Common Commands

```bash
# Install dependencies
bun install

# Start development server
bun run start

# Run on iOS/Android
bun run ios
bun run android

# Type checking
bun run compile

# Linting
bun run lint        # Fix lint issues
bun run lint:check  # Check without fixing

# Testing
bun run test            # Run all tests
bun run test:watch      # Watch mode
bun run test:maestro    # E2E tests with Maestro

# Build commands (creates local builds)
bun run build:ios:sim       # iOS simulator
bun run build:ios:device    # iOS device
bun run build:android:sim   # Android emulator
bun run build:android:device # Android device
```

## Architecture

### Entry Point
- `index.tsx` → `app/app.tsx` → `AppNavigator`

### Directory Structure
```
app/
├── components/     # Reusable UI components (Text, Button, Card, etc.)
├── config/         # App configuration
├── context/        # React contexts (AuthContext, ThemeContext)
├── i18n/           # Internationalization (i18next)
├── navigators/     # React Navigation setup
├── screens/        # Screen components
├── services/api/   # API client (apisauce)
├── theme/          # Design tokens, colors, typography
└── utils/          # Utility functions and hooks
```

### Key Patterns

**Path Aliases**: Use `@/` for app directory and `@assets/` for assets
```typescript
import { Button } from "@/components"
import logo from "@assets/images/logo.png"
```

**Navigation**: Uses React Navigation with native stack navigator. Auth state determines which screens are available in `AppNavigator.tsx`.

**State Persistence**: Uses `react-native-mmkv` for fast synchronous storage (auth tokens, theme preference, navigation state).

**Theming**: Full light/dark theme support via `ThemeProvider`. Use `useAppTheme()` hook to access current theme and the `themed()` helper for theme-aware styles.

**Components**: Use custom wrapper components instead of React Native primitives:
- Use `@/components/Text` instead of `react-native/Text`
- Use `@/components/Button` instead of `react-native/Button`
- Use `@/components/TextField` instead of `react-native/TextInput`
- Use `SafeAreaView` from `react-native-safe-area-context`

### Ignite Generator Anchors
When adding new screens, look for `IGNITE_GENERATOR_ANCHOR_APP_STACK_SCREENS` comments in navigator files.

## Code Style

- TypeScript strict mode enabled
- ESLint with Expo, Prettier, and import ordering rules
- Import order: react → react-native → expo → external → @/ aliases → relative
- Unused variables must be prefixed with `_`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magic-cabinet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/magic-cabinet)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## mx5

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `pnpm install` - Install dependencies (use pnpm, not npm)
- `pnpm expo start` - Start the development server
- `pnpm start` - Alias for expo start
- `pnpm android` - Run on Android emulator
- `pnpm ios` - Run on iOS simulator
- `pnpm web` - Run in web browser

### Running the app
Press `i` for iOS, `a` for Android, or scan QR code with Expo Go app

## Architecture

This is an Expo/React Native app for tracking annual mileage limits (10,000 km) for an MX-5. Key structure:

- **Navigation**: Uses React Navigation with bottom tabs (Dashboard, History) and modal screens (AddEntry)
- **State Management**: AsyncStorage for persistence, React state for UI
- **Data Model**:
  - `MileageEntry`: Individual odometer readings with date and optional notes
  - `YearlyData`: Groups entries by year
  - `AppConfig`: Stores initial setup date and optional starting odometer

## Key Components

- **Screens**: Setup flow → Main app (Dashboard/History tabs) → Add Entry modal
- **Storage**: All data operations in `src/utils/storage.ts` using AsyncStorage
- **Calculations**: Mileage tracking logic in `src/utils/calculations.ts`
- **Styling**: Dark theme with navy background (#1D3557) and red accent (#E63946)

## TypeScript

Strict mode enabled. No explicit lint/test commands configured.

---
> Source: [miermontoto/mx5](https://github.com/miermontoto/mx5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

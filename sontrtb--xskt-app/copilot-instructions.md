## xskt-app

> A comprehensive lottery application built with Expo and React Native, providing real-time results, predictions, dream book analysis, and lunar calendar features.

# XSKT App (Lottery Application)

A comprehensive lottery application built with Expo and React Native, providing real-time results, predictions, dream book analysis, and lunar calendar features.

## Project Overview

- **Framework:** [Expo](https://expo.dev/) (SDK 54) with [React Native](https://reactnative.dev/)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **Navigation:** [Expo Router](https://docs.expo.dev/router/introduction/) (using `Stack` and `Tabs`)
- **State Management:** [Zustand](https://github.com/pmndrs/zustand) with persistence
- **Data Fetching:** [TanStack Query (React Query)](https://tanstack.com/query/latest) & [Axios](https://axios-http.com/)
- **Styling:** React Native `StyleSheet` with a custom theme-based color system
- **Localization:** `i18n-js` supporting Vietnamese (vn), English (en), and Lao (lo)
- **Ads:** [react-native-google-mobile-ads](https://github.com/invertase/react-native-google-mobile-ads)
- **Firebase:** Integrated for Analytics and Cloud Messaging (FCM)

## Project Structure

- `api/`: API service definitions and axios configuration (`rootApi`).
- `app/`: Expo Router routes and layout definitions.
- `assets/`: Static assets including fonts (LexendDeca) and images.
- `components/`:
    - `commons/`: Reusable high-level components and layouts.
    - `ui/`: Fundamental UI library (Button, Text, Input, etc.).
- `configs/`: App configurations and static data (e.g., dream book data).
- `hooks/`: Custom React hooks (e.g., `useColor` for theme, `useSendMessage`).
- `lang/`: Internationalization translation files.
- `lib/`: Utility functions for dates, files, permissions, etc.
- `screen/`: Main screen implementations referenced by `app/` routes.
- `stores/`: Zustand stores for global state management.
- `theme/`: Theme definitions (colors, layout constants).
- `types/`: TypeScript type definitions.

## Development Conventions

### UI & Styling
- **Custom UI Components:** Always prefer using components from `components/ui/` (e.g., `TextUi`, `ButtonUi`, `TextInputUi`) to ensure consistent styling and theme support.
- **Theme Awareness:** Use the `useColor` hook to access theme-specific colors (primary, background, text, etc.) in your components.
- **Layout:** Utilize constants from `theme/layout.ts` (like `PADDING_PAGE`) for consistent spacing.

### State & Data
- **Global State:** Use Zustand stores in `stores/`. If the state needs to persist across app restarts, use the `persist` middleware with `AsyncStorage`.
- **API Calls:** Implement API requests in the `api/` directory using the `rootApi` utility.
- **Data Fetching:** Wrap API calls with `useQuery` or `useMutation` from TanStack Query for caching and state management.

### Navigation
- **Routes:** Add new screens by creating files in the `app/` directory.
- **Configuration:** Update `app/routers.tsx` to configure screen options (titles, headers, etc.) for the main stack.

### Internationalization
- **Translations:** Add new keys to `lang/vn.ts`, `lang/en.ts`, and `lang/lo.ts`.
- **Usage:** Use the translation helper (likely exported from `lang/i18n.ts`) to display localized strings.

## Building and Running

### Development
- `npm start`: Starts the Expo development server.
- `npm run android`: Runs the app on an Android emulator or connected device.
- `npm run ios`: Runs the app on an iOS simulator or connected device.

### Production
- `npx expo prebuild`: Generates the `android/` and `ios/` native directories.
- `eas build -p android --profile preview --local`: Builds a preview APK locally.
- `eas build -p android --profile production --local`: Builds a production AAB locally.

## Key Features
- **Live Results:** Real-time lottery results tracking.
- **Dream Book:** Keyword search for dream interpretation related to lottery numbers.
- **Lunar Calendar:** View auspicious dates and times.
- **Predictions/Forecast:** AI or algorithm-based lottery predictions.
- **Notes:** Personalized note-taking for tracking numbers.
- **Chat:** Assistant for dream interpretation and general lottery help.

---
> Source: [sontrtb/xskt-app](https://github.com/sontrtb/xskt-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

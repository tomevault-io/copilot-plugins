## trip-indo-mobile

> This is a React Native mobile application built with Expo for managing trips in Indonesia. It allows users to track their trips, manage participants, expenses, and destinations. The application uses Supabase for its backend, including database and authentication. The UI is built with React Native Elements and custom components, with a strong emphasis on a consistent design system. Navigation is handled by Expo Router.

# Project: TripMate Mobile App

## Project Overview

This is a React Native mobile application built with Expo for managing trips in Indonesia. It allows users to track their trips, manage participants, expenses, and destinations. The application uses Supabase for its backend, including database and authentication. The UI is built with React Native Elements and custom components, with a strong emphasis on a consistent design system. Navigation is handled by Expo Router.

## Building and Running

### Prerequisites

- Node.js and npm installed
- Expo CLI installed (`npm install -g expo-cli`)
- A Supabase project with the required tables (`trips`, `trip_participants`, `users`).
- A `.env` file at the root of the project with the following variables:
  ```
  EXPO_PUBLIC_SUPABASE_URL=<your-supabase-url>
  EXPO_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
  ```

### Key Commands

- **Install dependencies:**
  ```bash
  npm install
  ```

- **Run the application (development mode):**
  ```bash
  npm start
  ```
  This will open the Expo developer tools in your browser. You can then run the app on a simulator/emulator or on a physical device using the Expo Go app.

- **Run on a specific platform:**
  ```bash
  npm run ios
  npm run android
  ```

- **Run tests:**
  ```bash
  npm test
  ```

- **Build the application for different environments:**
  ```bash
  # Development build for iOS
  npm run build:dev

  # Preview build for iOS (e.g., for TestFlight)
  npm run build:preview

  # Production build for iOS
  npm run build:prod
  ```

- **Submit the app to the App Store:**
  ```bash
  npm run submit:ios
  ```

- **Deploy to TestFlight:**
  ```bash
  npm run deploy:testflight
  ```

## Development Conventions

- **State Management:** The application uses React Context for managing global state. Separate contexts are provided for authentication (`AuthContext`), trip data (`TripContext`), theme (`ThemeContext`), and notifications (`NotificationContext`).
- **Styling:** The application uses a combination of global stylesheets, NativeWind (Tailwind CSS for React Native), and a custom theme defined in `src/constants/theme.ts`.
- **Fonts:** Custom fonts are loaded in the root layout. The primary font family is "Ubuntu".
- **Services:** Business logic for interacting with the Supabase backend is encapsulated in services located in the `src/services` directory.
- **Typing:** The project is written in TypeScript, and types for the main data models are defined in the `src/types` directory.
- **Navigation:** The app uses Expo Router for file-based routing. The main navigation is a tab-based layout, with nested stack navigators for different features.
- **UI Components:** Reusable UI components are located in the `components` directory.
- **Linting and Formatting:** The project should be configured with ESLint and Prettier to maintain a consistent code style (though not explicitly found, it's a best practice).

---
> Source: [alexarts74/trip-indo-mobile](https://github.com/alexarts74/trip-indo-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->

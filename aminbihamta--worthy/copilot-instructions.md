## worthy

> Worthy is an offline-first mobile expense tracking application built with Expo, React Native, TypeScript, SQLite, and NativeWind. It allows users to track expenses, incomes, and transfers without an internet connection. Key features include manual account and category management, budgeting, savings goals, a receipt inbox with local image storage, and data insights with charts. The application supports both light and dark modes.

# Project: Worthy

## Project Overview

Worthy is an offline-first mobile expense tracking application built with Expo, React Native, TypeScript, SQLite, and NativeWind. It allows users to track expenses, incomes, and transfers without an internet connection. Key features include manual account and category management, budgeting, savings goals, a receipt inbox with local image storage, and data insights with charts. The application supports both light and dark modes.

## Building and Running

### Setup

1.  Install dependencies:
    ```bash
    npm install
    ```

2.  Start the development server:
    ```bash
    npm run start
    ```

### Running on a device/simulator

*   **iOS:**
    ```bash
    npm run ios
    ```
*   **Android:**
    ```bash
    npm run android
    ```

### Linting and Formatting

*   **Lint:**
    ```bash
    npm run lint
    ```
*   **Format:**
    ```bash
    npm run format
    ```

## Development Conventions

### Architecture

The project follows a modular architecture:

-   `src/db`: Contains all SQLite-related logic, including migrations, repositories, and data seeding.
-   `src/screens`: Contains all the application screens, organized by feature.
-   `src/components`: Contains shared and reusable UI components.
-   `src/utils`: Contains utility functions for money, date, and life-cost calculations.
-   `src/state`: Contains global state management using Zustand.
-   `src/theme`: Contains theme tokens and navigation theme configuration.
-   `src/navigation`: Contains the root navigator and defines the app's screen structure.

### Offline Data

The application is designed to be fully functional offline. All data is stored locally in a SQLite database using `expo-sqlite`. There are no network dependencies for any of the core features.

### Styling

The project uses [NativeWind](https://www.nativewind.dev/) for styling, which allows for using Tailwind CSS utility classes in React Native. The `tailwind.config.js` file defines the color palette, fonts, and other design tokens. The application supports both light and dark modes.

### Fonts

The application uses the Manrope font from Google Fonts, loaded via `@expo-google-fonts/manrope`.

### State Management

Global UI and settings state is managed with [Zustand](https://github.com/pmndrs/zustand).

### Navigation

Navigation is handled by [React Navigation](https://reactnavigation.org/). The main navigation is a bottom tab bar, with each tab having its own stack navigator.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AminBihamta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

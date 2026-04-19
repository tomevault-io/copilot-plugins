## presentx

> The frontend is a React Native application built with Expo, supporting both native (iOS/Android) and web platforms. It uses Expo Router for file-based navigation.

# App Architecture (`packages/app`)

The frontend is a React Native application built with Expo, supporting both native (iOS/Android) and web platforms. It uses Expo Router for file-based navigation.

**Key Files & Directories:**

*   **`app/`**: This directory contains all the screens and layouts managed by Expo Router.
    *   **`_layout.tsx` ([packages/app/app/_layout.tsx](mdc:packages/app/app/_layout.tsx))**: The root layout file. Sets up the main Stack navigator, applies global styles/providers (like `PaperProvider`, `SafeAreaProvider`, `GestureHandlerRootView`), and defines default screen options.
    *   **`index.tsx` ([packages/app/app/index.tsx](mdc:packages/app/app/index.tsx))**: The initial screen (mapped to the `/` route). Serves as the "Join Screen" where users enter an access code.
    *   **`presentation/[id].tsx` ([packages/app/app/presentation/[id].tsx](mdc:packages/app/app/presentation/[id].tsx))**: A dynamic route screen that displays the presentation content to the audience. It uses the `id` parameter from the URL, connects to WebSockets, listens for updates, and renders the current page content using specific viewer components.
*   **`src/`**: Contains core application logic, components, and utilities.
    *   **`api/client.ts` ([packages/app/src/api/client.ts](mdc:packages/app/src/api/client.ts))**: Configures an Axios instance for making HTTP requests to the backend API.
    *   **`lib/socket.ts` ([packages/app/src/lib/socket.ts](mdc:packages/app/src/lib/socket.ts))**: Manages the Socket.IO client connection, including connection/disconnection logic, event listeners, and functions for joining/leaving presentation rooms.
    *   **`store/presentationStore.ts` ([packages/app/src/store/presentationStore.ts](mdc:packages/app/src/store/presentationStore.ts))**: A Zustand store holding the global state related to the currently joined presentation (data, current page, audience count, connection status, loading/error states).
    *   **`components/viewers/`**: Contains components responsible for rendering different page types (e.g., `MultiChoiceViewer`, `RatingViewer`). These are used by `presentation/[id].tsx`.

**UI & State Management:**

*   **UI Library:** React Native Paper.
*   **State Management:** Zustand (`presentationStore`).
*   **Navigation:** Expo Router.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/msmygit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## my-dashboard-frontend

> This is a client-side web application built with Vue.js 3 and Vite. It serves as a personal dashboard with features like weather, news, a diary, and more. The application uses Vue Router for navigation, Pinia for state management, and Ant Design Vue for the UI component library. It includes a complete authentication system (login, registration, password updates) that communicates with a backend API via token-based authentication.

# GEMINI.md

## Project Overview

This is a client-side web application built with Vue.js 3 and Vite. It serves as a personal dashboard with features like weather, news, a diary, and more. The application uses Vue Router for navigation, Pinia for state management, and Ant Design Vue for the UI component library. It includes a complete authentication system (login, registration, password updates) that communicates with a backend API via token-based authentication.

## Building and Running

### Development

To run the app in development mode with hot-reloading:

```bash
npm run dev
```

### Build

To build the app for production:

```bash
npm run build
```

### Preview

To preview the production build locally:

```bash
npm run preview
```

### Deploy

To deploy the `dist` directory to GitHub Pages:

```bash
npm run deploy
```

## Development Conventions

- **Framework:** Vue.js 3 with the `<script setup>` syntax.
- **Build Tool:** Vite.
- **Routing:** `vue-router` is used for client-side routing. Routes are defined in `src/router/index.js` and protected by authentication guards.
- **State Management:** Pinia is used for global state management. The primary store is located at `src/stores/auth.js` and handles all authentication and user data. State is persisted in `localStorage`.
- **UI Library:** Ant Design Vue provides the UI components.
- **API Communication:** `axios` is used for making HTTP requests to the backend API. The API URL is configured via environment variables (`VITE_API_URL_LOCAL` for development and `VITE_API_URL_LIVE` for production).
- **Project Structure:**
  - `src/views`: Contains the top-level page components.
  - `src/components`: Contains reusable UI components.
  - `src/stores`: Contains Pinia state management modules.
  - `src/router`: Contains routing configuration and navigation guards.
- **Styling:** Global styles are in `src/style.css`, and component-specific styles are within the `.vue` files, sometimes scoped. The project also uses Tailwind CSS for utility classes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skwongman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/skwongman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

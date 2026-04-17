## side-my-tools

> This document provides a comprehensive overview of the `side-my-tools` project, its architecture, and development practices to be used as instructional context for future interactions.

# GEMINI.md: Project Context

This document provides a comprehensive overview of the `side-my-tools` project, its architecture, and development practices to be used as instructional context for future interactions.

## Project Overview

This is a full-stack application designed as a **Telegram Mini App** for personal inventory management, specifically for food items. The project uses a modern web stack with a React frontend and a Supabase backend.

### Core Functionality

- **Food Inventory Management**: Users can add, view, update, and delete food items in their inventory. Each item can have a name, quantity, expiration date, category, and an associated image.
- **Telegram Integration**: The application is designed to run inside Telegram as a Mini App. User authentication is handled via a custom Telegram bot.
- **User Authentication**: A Telegram bot manages user registration. When a user interacts with the `/login` command, a new user account is created in Supabase Auth, linking their Telegram ID to an application user profile.
- **User Settings**: The application includes a page for users to manage their settings, including a field for a Gemini API key, suggesting future or existing integration with Google's Gemini models.

### Architecture & Tech Stack

- **Frontend**:
  - **Framework**: React 19 with Vite and TypeScript.
  - **UI**: Built with Tailwind CSS and a set of `shadcn/ui`-like components using Radix UI primitives (`@radix-ui/*`) and `lucide-react` for icons.
  - **State Management**: A mix of `TanStack Query` for server state (caching, refetching) and `Zustand` for global client state.
  - **Forms**: `React Hook Form` with `Zod` for schema-based validation.
  - **Routing**: `react-router`.
  - **Telegram Integration**: `@tma.js/sdk-react` is used to integrate the web app with the Telegram Mini App platform.

- **Backend (Supabase)**:
  - **Database**: PostgreSQL with tables for `users`, `user_settings`, and `food_items`.
  - **Authentication**: Supabase Auth is used to manage user identities, linked to Telegram accounts.
  - **Serverless Functions**: Deno-based Edge Functions handle backend logic. A key function is the `telegram-bot` which manages user interactions and registration.
  - **Storage**: Supabase Storage is used to store images for food items in a dedicated `food-images` bucket.

## Building and Running

The project is a standard Vite application. Key scripts are defined in `package.json`.

- **Install Dependencies**:
  ```bash
  npm install
  ```

- **Run Development Server**:
  Starts the Vite dev server, typically on `http://localhost:5173`.
  ```bash
  npm run dev
  ```
  To test the Telegram Mini App authentication in a browser, use the `tgWebAppData` query parameter as described in `README.md`.

- **Build for Production**:
  Transpiles the TypeScript code and bundles the application for deployment. The output is placed in the `dist/` directory.
  ```bash
  npm run build
  ```

- **Linting**:
  Runs ESLint to check for code quality and style issues.
  ```bash
  npm run lint
  ```

- **Preview Production Build**:
  Serves the `dist/` directory to preview the production build locally.
  ```bash
  npm run preview
  ```

## Development Conventions

- **Styling**: The project uses Tailwind CSS for utility-first styling. Custom components are built following the patterns of `shadcn/ui`.
- **Component Structure**: UI components are located in `src/components/ui`, while feature-specific components (e.g., for Inventory) are co-located with their respective features under `src/pages/Inventory/components`.
- **State Management**: Server state (data fetching, mutations) should be handled by `TanStack Query` via custom hooks (e.g., `useInfiniteQuery`, `useUpdateFoodItem`). Global client-side state is managed with `Zustand` stores (e.g., `src/stores/auth.store.ts`).
- **Backend Functions**: Server-side logic is implemented as Deno-based TypeScript functions in the `supabase-functions/` directory. Environment variables (e.g., Supabase keys, bot tokens) are required to run these functions.
- **Database Schema**: The database structure is documented in `docs/database-structure.md`. Any changes to the database should be reflected in this document.
- **API Interaction**: The frontend communicates with the Supabase backend using the `supabase-js` client, which is configured in `src/lib/supabaseClient.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quoclucnguyen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

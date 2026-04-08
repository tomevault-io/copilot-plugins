## llmeta-client-prototype

> This is a Next.js project for a metaverse experience, leveraging AI for new communication methods. It's built with React and TypeScript, using `@react-three/fiber` for 3D rendering and `colyseus.js` for real-time multiplayer functionality. The application supports both desktop and XR modes.

# Project: LLM Metaverse Client

## Project Overview

This is a Next.js project for a metaverse experience, leveraging AI for new communication methods. It's built with React and TypeScript, using `@react-three/fiber` for 3D rendering and `colyseus.js` for real-time multiplayer functionality. The application supports both desktop and XR modes.

### Key Technologies:

- **Framework:** Next.js
- **Language:** TypeScript
- **3D Rendering:** `@react-three/fiber`, `@react-three/drei`
- **Multiplayer:** `colyseus.js`
- **UI:** Radix UI, Tailwind CSS
- **XR:** `@react-three/xr`

### Architecture:

- **`src/app`:** Contains the main application pages, including the home page (`/`), lobby (`/lobby`), experience (`/experience`), and VRM avatar preview page (`/vrm`).
- **`src/components`:** Reusable React components, organized by feature:
  - `experience/`: Experience-specific components
  - `main/`: Main page components
  - `player/`: Player-related components
  - `scene/`: 3D scene components (Level, SceneMap)
  - `ui/`: UI components (Shadcn/ui)
  - `vrm/`: VRM avatar components
- **`src/hooks`:** Custom React hooks (camelCase naming convention) for managing side effects, such as the Colyseus connection lifecycle (`useColyseusLifecycle.ts`), realtime presence (`useRealtimePresenceRoom.ts`), and XR support detection (`useXrSupport.ts`).
- **`src/types`:** Centralized TypeScript type definitions:
  - `player.ts`: Player and multiplayer-related types
  - `user.ts`: User and UI-related types
- **`src/stores`:** Zustand stores for global state management.
- **`src/utils`:** Utility functions, including the Colyseus client setup (`colyseus.ts`).
- **`src/lib`:** Library configurations (e.g., Supabase client).
- **`public`:** Static assets, including 3D models (`.glb`, `.vrm`) and fonts.

## Building and Running

### Development:

To run the development server:

```bash
yarn dev
```

This will start the application on [http://localhost:3000](http://localhost:3000).

To run the development server with HTTPS (required for some XR features):

```bash
yarn dev:https
```

### Production:

To build the application for production:

```bash
yarn build
```

To start the production server:

```bash
yarn start
```

### Linting:

To lint the codebase:

```bash
yarn lint
```

## Development Conventions

- **Naming Conventions:**
  - Hook files: camelCase (e.g., `useColyseusLifecycle.ts`, `useRealtimePresenceRoom.ts`)
  - Component files: PascalCase (e.g., `AvatarStack.tsx`, `RemotePlayer.tsx`)
  - Type files: PascalCase (e.g., `player.ts`, `user.ts`)
  - Utility files: camelCase (e.g., `colyseus.ts`, `utils.ts`)
- **Styling:** The project uses Tailwind CSS for styling, with components from Radix UI (Shadcn/ui).
- **State Management:** Component-level state is managed with React hooks. Global state for the multiplayer experience is managed by Colyseus. Application-wide state uses Zustand stores.
- **3D:** The 3D scenes are built with `@react-three/fiber` and `@react-three/drei`, following a declarative, component-based approach.
- **Multiplayer:** The connection to the Colyseus server is managed by the `useColyseusLifecycle` hook, which automatically connects and disconnects from the room.
- **Type Definitions:** Shared types are centralized in the `src/types` directory for better maintainability and reusability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ut42tech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ut42tech)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

## storycraft

> StoryCraft is an AI-powered video storyboard generation platform that transforms text descriptions into complete video narratives. It leverages Google's generative AI models (Imagen, Veo, Chirp, Lyria) to create cinematic scenes, voiceovers, and music.

# StoryCraft

## Project Overview

StoryCraft is an AI-powered video storyboard generation platform that transforms text descriptions into complete video narratives. It leverages Google's generative AI models (Imagen, Veo, Chirp, Lyria) to create cinematic scenes, voiceovers, and music.

**Key Technologies:**

- **Framework:** Next.js 15 (App Router) with React 18
- **Language:** TypeScript
- **Styling:** Tailwind CSS (Custom Palette), Framer Motion, Radix UI
- **State Management:** Zustand (Client), TanStack Query (Server), React Context
- **AI & Cloud:** Google Vertex AI, Google Cloud Storage, Firestore, Google Cloud Text-to-Speech
- **Video Processing:** FFmpeg (server-side)

## Build, Run, and Test

### scripts

- **Development Server:** `npm run dev` (Starts at http://localhost:3000)
- **Build for Production:** `npm run build`
- **Start Production Server:** `npm run start`
- **Lint Code:** `npm run lint`
- **Format Code:** `npm run format`
- **Type Check:** `npm run check`

### Testing

- **Unit Tests:** `npm run test` (uses Vitest)
- **UI Tests:** `npm run test:ui` (Vitest UI)
- **Coverage:** `npm run test:coverage`
- **E2E Tests:** `npm run test:e2e` (uses Playwright)

## Architecture & Structure

The project follows a **feature-based** architecture, organizing code by domain rather than type.

### Key Directories

- **`app/features/`**: Contains self-contained feature modules.
    - `create/`: Story creation interface.
    - `editor/`: Timeline-based video editor.
    - `scenario/`: Story scenario management.
    - `storyboard/`: Scene editing.
    - `stories/`: Story management.
    - `shared/`: Shared feature logic.
    - _Structure within features:_ `components/`, `hooks/`, `actions/`, `stores/`, `types/`.
- **`app/actions/`**: Server Actions for data mutations and AI operations.
- **`app/api/`**: API Route Handlers (standardized responses).
- **`lib/`**: Core utilities.
    - `api/`: Wrappers for AI services (gemini.ts, imagen.ts, etc.).
    - `storage/`: GCS and Firestore integrations.
    - `utils/`: Helpers (ffmpeg.ts, env.ts, etc.).
- **`components/ui/`**: Reusable UI components (buttons, dialogs, etc.).

## Development Conventions

- **Path Aliases:** Use `@/` to refer to the project root (e.g., `import ... from "@/lib/utils"`).
- **Styling:** Tailwind CSS with custom colors:
    - **Primary:** `#008DD5`
    - **Secondary:** `#373F51`
    - **Accent:** `#F56476`
- **State:**
    - Use **Zustand** for complex client-side UI state (e.g., timeline editor).
    - Use **TanStack Query** for server data fetching and caching.
- **Testing:**
    - Place unit tests next to the file being tested or in `__tests__`.
    - Use `.test.ts` or `.test.tsx` extensions.
    - E2E tests are located in `e2e/`.

## Environment Variables

The application requires a `.env.local` file with the following key configurations:

- **Google Cloud:** `PROJECT_ID`, `LOCATION`, `GCS_VIDEOS_STORAGE_URI`, `FIRESTORE_DATABASE_ID`
- **Authentication:** `AUTH_SECRET`, `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`, `NEXTAUTH_URL`
- **App Config:** `MODEL`, `USE_COSMO`, `LOG_LEVEL`

## General Instructions

- For every task : run `npm run preflight` to verify the code quality; fix errors and re-run to verify you didn't introduced nex problems; Do that until everything is fixed.
- Do not use git unless explicitly asked to.

---
> Source: [mblanc/storycraft](https://github.com/mblanc/storycraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

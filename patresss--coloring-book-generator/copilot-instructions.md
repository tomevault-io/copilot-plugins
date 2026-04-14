## coloring-book-generator

> This is a full-stack application for generating, viewing, and printing coloring pages.

# Project Overview

This is a full-stack application for generating, viewing, and printing coloring pages.

**Frontend:**
- React with Vite
- Material-UI for components
- Communicates with the backend via a REST API

**Backend:**
- Node.js with Express
- TypeScript
- Uses Gemini for image generation and OpenAI for transcription and prompt improvement.
- Saves generated images and metadata to the filesystem.

**Containerization:**
- Docker and Docker Compose are used to run the frontend and backend services in containers.

# Building and Running

## Local Development

**Backend:**
```bash
cd backend
npm install
npm run dev
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

## Docker Compose

To run the application using Docker Compose:

```bash
docker-compose up -d
```

This will start the backend and frontend services. The frontend will be available at `http://localhost:5173`.

# Development Conventions

- **TypeScript:** The entire project is written in TypeScript with strict mode enabled.
- **Package Manager:** The project uses `npm` for package management.
- **Code Style:**
    - 2-space indentation.
    - Semicolons are used.
    - Backend files are lowercase (e.g., `openai.ts`).
    - React components are PascalCase (e.g., `App.tsx`).
- **Validation:** The backend uses `zod` for data validation.
- **UI:** The frontend uses Material-UI for its components.
- **Containerization:** The project uses `ghcr.io` for hosting Docker images.

# Testing

- No test harness is configured yet.
- If adding tests:
    - **Frontend:** Use Vitest + React Testing Library, with test files named `*.test.tsx` and colocated with the source files.
    - **Backend:** Use Vitest/Jest + supertest, with test files named `*.test.ts` under the `src/` directory.

# Environment Variables

- Copy `.env.example` to `.env` in the root directory or the `backend/` directory.
- Key environment variables:
    - `OPENAI_API_KEY`
    - `GEMINI_API_KEY`
    - `PRINTER_URI`
    - `DATA_DIR`
    - `GEMINI_IMAGE_MODELS`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Patresss)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Patresss)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

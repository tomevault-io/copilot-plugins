## diam-ai-pilot

> This document provides essential context for Qwen Code / Gemini CLI / Kiro to understand and assist with the DIAM AI Pilot project, an educational AI prototype built with a React frontend and a TypeScript Node.js/Express backend, utilizing the Google Qwen API.

# DIAM AI Pilot - Project Context for Qwen Code

This document provides essential context for Qwen Code / Gemini CLI / Kiro to understand and assist with the DIAM AI Pilot project, an educational AI prototype built with a React frontend and a TypeScript Node.js/Express backend, utilizing the Google Qwen API.

## Project Overview

* **Name**: DIAM AI Pilot / BridgEdu
* **Purpose**: An educational AI assistant prototype for the DIAM course at ISCTE. It allows students to upload PDF documents (like course materials) and ask questions about them. The AI processes the document content and provides answers based on that context.
* **Core Technologies**:
  * **Frontend**: React (JavaScript/TypeScript)
  * **Backend**: Node.js, Express, TypeScript
  * **AI Service**: Google Qwen API
  * **Key Libraries**: `axios` (HTTP client), `multer` (file uploads), `pdf-parse` (PDF processing), `uuid` (session IDs)
* **Architecture**: A monorepo structure with separate `frontend` and `backend` directories, orchestrated by a root `package.json` using NPM workspaces. The frontend communicates with the backend via REST API calls.

## Project Structure

```
.
├── backend/                 # Node.js/Express backend
│   ├── src/                 # TypeScript source files
│   │   ├── config/          # Configuration (environment variables)
│   │   ├── middleware/      # Express middleware (CORS, error handling, uploads, monitoring, rate limiting)
│   │   ├── routes/          # API route definitions
│   │   ├── services/        # Core business logic (PDF processing, Qwen client, error service, logger)
│   │   ├── types/           # TypeScript type definitions
│   │   └── server.ts        # Main application entry point
│   ├── package.json         # Backend dependencies and scripts
│   ├── tsconfig.json        # TypeScript configuration
│   └── ecosystem.config.js  # PM2 deployment configuration
├── frontend/                # React frontend
│   ├── src/                 # React source files
│   │   ├── components/      # React components (UI building blocks)
│   │   ├── services/        # Frontend services (API calls)
│   │   ├── types/           # TypeScript type definitions
│   │   ├── utils/           # Utility functions
│   │   ├── index.tsx        # Main entry point
│   │   └── components/App.tsx # Main application component
│   └── package.json         # Frontend dependencies and scripts
├── scripts/                 # Utility scripts (e.g., performance testing)
├── Planning/                # Project planning documents
├── package.json             # Root package.json for workspaces and global scripts
├── .env.example             # Example environment variables file
├── .gitignore               # Git ignore rules
└── Dockerfile              # Docker configuration for deployment
```

## Building and Running

### Prerequisites

* Node.js (version likely 18.x or 20.x based on dependencies)
* NPM (comes with Node.js)
* A Google Qwen API key (for backend functionality)

### Environment Setup

1.  Copy the `.env.example` file in the project root (and potentially in `backend/`) to `.env`.
2.  Fill in the required environment variables, especially `GEMINI_API_KEY`, in the `.env` file(s).

### Development Commands

* `npm run install:all`: Installs dependencies for the root, frontend, and backend projects.
* `npm run dev:frontend`: Starts the React development server (usually on `http://localhost:3000`).
* `npm run dev:backend`: Starts the Express development server using `ts-node-dev` (usually on `http://localhost:3001`).
* `npm run dev:all`: Starts both the frontend and backend development servers concurrently.
* `npm run test`: Runs tests for both the frontend and backend.
* `npm run lint`: Runs the linter for both the frontend and backend (note: linting appears to be a TODO).
* `npm run build`: Builds both the frontend (React) and backend (TypeScript to JavaScript) for production.

### Production Commands

* `npm run build:prod`: Builds the project for production.
* `npm run start:prod`: Starts the built backend application using `node`.

## Key Files and Components

### Backend (`backend/`)

* **Main Server**: `src/server.ts` - Entry point, sets up Express app, middleware, routes, and starts the server.
* **Configuration**: `src/config/environment.ts` - Loads environment variables and defines the `env` object used throughout the backend.
* **API Routes**: `src/routes/api.ts` - Defines the `/api/ask` endpoint (core functionality), `/api/rate-limit-status`, and `/api/health`.
* **Key Services**:
  * `src/services/geminiClient.ts` (inferred from usage) - Handles communication with the Google Qwen API.
  * `src/services/pdfProcessor.ts` (inferred from usage) - Processes uploaded PDF files.
  * `src/services/errorService.ts` (inferred from usage) - Centralizes error handling and formatting.
* **Middleware**:
  * `src/middleware/upload.ts` - Handles file uploads using `multer`.
  * `src/middleware/rateLimiter.ts` - Implements rate limiting for API requests.
  * `src/middleware/errorHandler.ts` - Catches and formats errors for API responses.
  * `src/middleware/cors.ts` - Configures Cross-Origin Resource Sharing.
  * `src/middleware/monitoring.ts` - Adds request IDs and performance tracking.

### Frontend (`frontend/`)

* **Main Application**: `src/components/App.tsx` - The core React component managing application state (file, question, response, UI state) and rendering the main layout (header, left nav, center content, right rail).
* **Core Components**:
  * `src/components/FileUpload.tsx` (inferred from usage) - Component for file selection and upload (though upload is primarily handled via the header in `App.tsx`).
  * `src/components/QuestionInput.tsx` (inferred from usage) - Component for entering questions.
  * `src/components/ResponseDisplay.tsx` (inferred from usage) - Component for displaying AI responses.
* **Services**:
  * `src/services/api.ts` - Contains the `askQuestion` function that sends requests to the backend `/api/ask` endpoint.
  * `src/services/errorService.ts` (inferred from usage) - Handles frontend error processing and user-facing messages.
* **Entry Point**: `src/index.tsx` - Renders the main `App` component.

## Development Conventions

* **Language**: TypeScript is used for both frontend and backend.
* **Frontend Styling**: CSS modules or plain CSS files (e.g., `App.css`) are used, with a design inspired by Moodle's layout.
* **Backend API**: RESTful API design using Express.js.
* **Error Handling**: Centralized error handling services exist on both frontend and backend.
* **Testing**: Jest is configured for backend testing (`backend/jest.config.js`), and React Testing Library is included for frontend testing.
* **Environment Variables**: Managed via `dotenv` and a central configuration file (`environment.ts`) on the backend.
* **Logging**: Custom logging service (`logger.ts`) is used on the backend.
* **Rate Limiting**: Implemented on the backend API (`/api/ask`) to prevent abuse.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/santyveloso)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/santyveloso)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

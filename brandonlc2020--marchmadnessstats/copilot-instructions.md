## marchmadnessstats

> This document provides architectural overview, development workflows, and technical context for the **March Madness Stats** project, a full-stack NCAA Men's College Basketball statistics application.

# March Madness Stats 🏀 - Gemini Context

This document provides architectural overview, development workflows, and technical context for the **March Madness Stats** project, a full-stack NCAA Men's College Basketball statistics application.

## Project Overview

March Madness Stats is a data-driven application that aggregates and visualizes NCAA basketball data. It follows a decoupled architecture:
- **Backend**: A Spring Boot "Edge Service" that proxies and orchestrates data from the external [College Basketball Data API](https://collegebasketballdata.com/).
- **Frontend**: A React-based Single Page Application (SPA) that provides interactive dashboards, historical rankings, and AI-powered statistical analysis.

### Core Technologies
- **Backend**: Java 21, Spring Boot 3, Spring WebFlux (Reactive), Maven, OpenAPI Generator.
- **Frontend**: React 19, TypeScript, Material-UI (MUI) 7, Plotly.js (Data Vis), Firebase (Favorites/Persistence), Google Gemini (AI Analysis).

---

## Directory Structure

```text
/
├── march-madness-stats/
│   ├── backend/           # Spring Boot application
│   │   ├── src/main/java  # Source code (API, Service, Config)
│   │   ├── src/main/resources/schemas/swagger.json # Source for OpenAPI generation
│   │   └── pom.xml        # Maven configuration
│   └── frontend/          # React application
│       ├── src/           # Components, hooks, views, types
│       └── package.json   # Node.js dependencies and scripts
└── README.md              # General project documentation
```

---

## Backend Development (Spring Boot)

### API Generation
The backend uses an **OpenAPI-first** approach. API interfaces and models are generated from the `swagger.json` file located in `backend/src/main/resources/schemas/`.
- Generated code resides in `target/generated-sources`.
- Do not manually edit files in `com.blc.backend.api` (interfaces) or `com.blc.backend.model` (POJOs) if they are part of the generated package.

### Key Components
- **`CbbApiService`**: The core service using Spring `WebClient` to fetch data from the external API. It uses reactive `Flux` and `Mono` types.
- **Controllers**: Located in `com.blc.backend.api`, these implement the generated interfaces to expose endpoints to the frontend.
- **Configuration**: `JacksonConfig` and `WebConfig` handle JSON serialization and CORS.

### Commands
- **Build & Generate**: `mvn clean install` (Run this first to generate OpenAPI models).
- **Run**: `mvn spring-boot:run`.
- **Test**: `mvn test`.

---

## Frontend Development (React)

### State & Data Fetching
- **Hooks**: Data fetching is encapsulated in custom hooks (e.g., `useTeams`, `useGames`) found in `src/hooks/`.
- **API Helper**: `src/hooks/apiHelper.ts` provides a unified fetching utility that respects the `LOCAL_BASE_URL`.
- **Types**: Type definitions for API responses are maintained in `src/types/api.ts`.

### Visualizations & AI
- **Plotly**: Used for complex charts like `ShotDistributionPieChart` and `ShotTypeBarChart`.
- **Gemini**: The `GeminiAnalysis` component leverages the `@google/generative-ai` SDK to provide contextual insights.

### Commands
- **Setup**: `npm install`.
- **Run**: `npm start` (Runs on `http://localhost:3000`).
- **Build**: `npm run build`.

---

## Environment Configuration

Both tiers require environment variables. Templates can be found in `sample.env` files within each directory.

### Backend (`.env` or system vars)
- `CBB_API_KEY`: Required for authorizing requests to the College Basketball Data API.

### Frontend (`.env`)
- `LOCAL_BASE_URL`: Usually `http://localhost:8080`.
- `FIREBASE_*`: Configuration for the Firebase project.
- `GEMINI_API_KEY`: API key for Google Gemini analysis features.

---

## Development Conventions

1. **Reactive Patterns**: Always prefer `Flux`/`Mono` in the backend service layer to maintain the non-blocking nature of the Edge Service.
2. **Type Safety**: Maintain strict TypeScript interfaces in the frontend. If the backend API changes, update the `swagger.json`, rebuild the backend, and update `src/types/api.ts` accordingly.
3. **Component Styling**: Use Material-UI `SxProps` or styled-components via Emotion (MUI default) for styling. Avoid global CSS where possible.
4. **Testing**: Add unit tests for new services in the backend and utilize `react-testing-library` for critical frontend components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrandonLC2020)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BrandonLC2020)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

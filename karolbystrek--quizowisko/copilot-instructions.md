## quizowisko

> This document provides essential context for AI models interacting with this project. Adhering to these guidelines will ensure consistency and maintain code quality.

# GEMINI.MD: AI Collaboration Guide

This document provides essential context for AI models interacting with this project. Adhering to these guidelines will ensure consistency and maintain code quality.

## 1. Project Overview & Purpose

* **Primary Goal:** A modern, interactive quiz application ("Quizowisko") designed for students to upload, manage, and study questions. It emphasizes a unique, hand-drawn "sketch" aesthetic for its user interface.
* **Business Domain:** EdTech / Personal Learning Tools.

## 2. Core Technologies & Stack

* **Languages:** TypeScript (Strict mode enabled).
* **Frameworks & Runtimes:** 
    * **Runtime:** Bun (Latest) - used for serving, bundling, and package management.
    * **Frontend:** React 19 (Functional components, Hooks).
* **UI & Styling:**
    * **CSS:** Tailwind CSS 4.
    * **Components:** shadcn/ui (New York style), Lucide React for icons.
    * **Special Effects:** `roughjs` for the hand-drawn canvas-based UI elements.
    * **Animation:** GSAP (GreenSock Animation Platform).
* **Package Manager:** Bun (`bun.lockb`).

## 3. Architectural Patterns

* **Overall Architecture:** A full-stack Bun application. Bun serves as the web server (handling API routes and static file serving) and also acts as the bundler for the React SPA.
* **Directory Structure Philosophy:**
    * `/src`: Root of source code.
        * `index.ts`: Server entrypoint (Bun.serve).
        * `frontend.tsx`: Client-side React entrypoint.
        * `components/`: UI components, divided into business logic components and primitive `ui/` components.
        * `components/ui/`: Contains shadcn/ui primitives and custom aesthetic components like `RoughBox.tsx`.
        * `hooks/`: Custom React hooks (e.g., `useQuiz`, `useThemeObserver`).
        * `data/`: Static data and JSON files (e.g., `questions.json`).
        * `types/`: Global TypeScript interfaces and types.
        * `lib/`: Utility functions (e.g., `cn` for Tailwind class merging).
    * `styles/`: Global CSS and Tailwind configuration.
    * `assets/`: Static assets like fonts (`Excalifont`).

## 4. Coding Conventions & Style Guide

* **Formatting:** Standard TypeScript/React formatting. Indentation: 2 spaces.
* **Naming Conventions:**
    * `variables`, `functions`: camelCase (`currentQuestion`, `handleAnswer`)
    * `classes`, `components`: PascalCase (`QuizApp`, `RoughBox`)
    * `files`: PascalCase for components (`DotGrid.tsx`), camelCase or kebab-case for utilities.
* **UI Consistency:** 
    * Use the `RoughBox` component for containers to maintain the hand-drawn aesthetic.
    * Leverage `shadcn/ui` components but wrap or style them to match the "sketch" theme where appropriate.
* **Path Aliases:** Use the `@/*` alias for imports (mapped to `./src/*`).

## 5. Key Files & Entrypoints

* **Server Entrypoint:** `src/index.ts` - Configures `Bun.serve` with routes and Hot Module Reloading (HMR).
* **Frontend Entrypoint:** `src/frontend.tsx` - Mounts the React application to the DOM.
* **Build Script:** `build.ts` - A custom Bun script for bundling the application for production using `Bun.build` and `bun-plugin-tailwind`.
* **Configuration:** 
    * `components.json`: shadcn/ui configuration.
    * `bunfig.toml`: Bun-specific configuration.
    * `tsconfig.json`: TypeScript configuration with strict checks.

## 6. Development & Testing Workflow

* **Local Development:** Run `bun dev` to start the Bun server with HMR. The server handles both the API and the frontend.
* **Build Process:** Run `bun run build` to generate the production-ready files in the `dist/` directory.
* **Production Start:** Run `bun start` to serve the application in production mode.

## 7. Specific Instructions for AI Collaboration

* **Hand-Drawn Style:** When creating new UI elements, prioritize using `RoughBox` or extending `roughjs` logic. The app uses `Excalifont-Regular` to complement this style.
* **Bun-First:** Prefer Bun's built-in APIs (e.g., `Bun.file`, `Bun.serve`, `Bun.password`) over Node.js equivalents where possible.
* **React 19:** Utilize React 19 features where appropriate, ensuring compatibility with the current setup.
* **Data Structure:** Adhere to the `Question` and `Answer` interfaces defined in `src/types/quiz.ts` when modifying question data.
* **Adding Dependencies:** Use `bun add <package>` and ensure any new CSS requirements are compatible with Tailwind 4.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karolbystrek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

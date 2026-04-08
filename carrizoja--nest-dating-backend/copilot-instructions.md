## nest-dating-backend

> **devmatch** is a server-side application built with the [NestJS](https://nestjs.com/) framework. It appears to be a dating platform specifically designed for developers.

# GEMINI.md - Project Context

## Project Overview
**devmatch** is a server-side application built with the [NestJS](https://nestjs.com/) framework. It appears to be a dating platform specifically designed for developers.

- **Framework:** NestJS v11.0.1
- **Language:** TypeScript
- **Architecture:** Modular architecture following NestJS conventions.
- **Key Modules:**
  - `AppModule`: The root module of the application.
  - `ProfilesModule`: Handles profile-related functionality.

## Core Technologies
- **Backend:** Node.js with NestJS.
- **Validation/Metadata:** `reflect-metadata`, `rxjs`.
- **Testing:** Jest for unit tests and Supertest for E2E testing.
- **Linting/Formatting:** ESLint and Prettier.

## Building and Running
Based on the `README.md` and `package-lock.json`, the following commands are available:

- **Install Dependencies:** `npm install`
- **Development Mode:** `npm run start:dev` (watch mode)
- **Production Start:** `npm run start:prod`
- **Build:** `npm run build`
- **Unit Tests:** `npm run test`
- **E2E Tests:** `npm run test:e2e`
- **Linting:** `npm run lint` (inferred from `eslint.config.mjs`)

## Development Conventions
- **Code Style:** Adheres to Prettier and ESLint configurations found in `.prettierrc` and `eslint.config.mjs`.
- **Naming:** Follows standard NestJS naming conventions:
  - Controllers: `*.controller.ts`
  - Services: `*.service.ts`
  - Modules: `*.module.ts`
  - Tests: `*.spec.ts` or `*.e2e-spec.ts`
- **Strict Typing:** TypeScript is used throughout with standard NestJS configurations.

## Project Structure
- `src/`: Main source code directory.
  - `app.module.ts`: Root application module.
  - `main.ts`: Application entry point.
  - `profiles/`: Contains logic for the Profiles module.
- `test/`: E2E test files and configurations.
- `.agents/`: Contains project-specific agent personas and guidelines.

## Known Issues / Observations
- **Missing `package.json`:** The root `package.json` file is currently missing from the directory, although `package-lock.json` and `node_modules` are present. This may cause issues with package management and script execution. The `package-lock.json` indicates that the project name is "devmatch" version "0.0.1".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carrizoja)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carrizoja)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

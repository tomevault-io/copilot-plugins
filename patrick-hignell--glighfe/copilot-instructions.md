## glighfe

> This is a full-stack TypeScript project that uses React for the frontend and Node.js/Express for the backend. It includes a simple API, frontend routing, and a database connection.

# Project Overview

This is a full-stack TypeScript project that uses React for the frontend and Node.js/Express for the backend. It includes a simple API, frontend routing, and a database connection.

## Key Technologies

-   **Frontend:** React, Vite, React Router, Tailwind CSS
-   **Backend:** Node.js, Express.js
-   **Database:** Knex.js with SQLite3
-   **Authentication:** Auth0
-   **Linting/Formatting:** ESLint, Prettier
-   **Testing:** Vitest, React Testing Library

## Project Structure

-   `client/`: Contains the frontend React application.
-   `server/`: Contains the backend Express server.
-   `models/`: Contains data models shared between the client and server.
-   `server/db/`: Contains database-related files, including migrations and seeds.
-   `server/routes/`: Contains the API routes for the server.

## Building and Running

### Available Scripts

-   `npm install`: Install all dependencies.
-   `npm run dev`: Prompt the user to run the development server themselves . Start the development servers for both the client and server.
-   `npm run build`: Build the client and server for production.
-   `npm start`: Start the production server.
-   `npm test -- --run`: Run the test suite.
-   `npm run lint`: Lint the project files.
-   `npm run knex`: Run Knex database commands. For example, to run the latest migrations, use `npm run knex migrate:latest`.

### Development

1.  Run `npm install` to install the dependencies.
2.  Run `npm run dev` to start the development server.
3.  The client will be available at `http://localhost:5173` and the server at `http://localhost:3000`.

### Production

1.  Run `npm run build` to build the project.
2.  Run `npm start` to start the production server.

## Development Conventions

-   **Code Style:** The project uses ESLint and Prettier to enforce a consistent code style. Run `npm run lint` to check for linting errors and `npm run format` to format the code.
-   **Testing:** The project uses Vitest and React Testing Library for testing. Test files are located alongside the files they are testing.
-   **Database:** Database migrations and seeds are managed with Knex. Use the `npm run knex` command to interact with the database.
-   **Authentication:** The project is set up to use Auth0 for authentication. You will need to configure your Auth0 domain, client ID, and audience in `client/index.tsx`.

## PromptKit Quick Reference
- Review the available artefacts when the student requests them:
  - Protocol: `promptkit/protocols/setup.md` — instructions for updating these CLI briefings.
  - Workflow: `promptkit/workflows/tutor.md` — guide for tutoring/explanation sessions.
  - Workflow: `promptkit/workflows/reflect.md` — guide for documenting outcomes and next steps.
- Student notes live in `promptkit/notes/`; The table in `progress-journal.md` is main place to update with reflections. Instructor Activities are in `promptkit/activities/` (read-only).
- When new workflows arrive, expect additional files under `promptkit/workflows/`.

---
> Source: [patrick-hignell/GlIGHFE](https://github.com/patrick-hignell/GlIGHFE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->

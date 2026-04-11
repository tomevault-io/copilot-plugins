## restaurant-example

> This document provides a comprehensive overview of "The Golden Spoon" restaurant website project, designed to guide AI-assisted development.

# Gemini Code Assistant Project Overview

This document provides a comprehensive overview of "The Golden Spoon" restaurant website project, designed to guide AI-assisted development.

## 1. Project Vision & Purpose

The project is a static, visually appealing, and high-performance demo restaurant website. Its primary goals are to allow customers to easily browse menus, check opening times, and find contact information. The architecture prioritizes speed, SEO, and a smooth user experience.

## 2. Core Technologies

-   **Static Site Generator**: [Astro](https://astro.build/) is used for its performance benefits and component-based architecture.
-   **Styling**: [TailwindCSS](https://tailwindcss.com/) provides a utility-first CSS framework for rapid and consistent styling.
-   **Language**: [TypeScript](https://www.typescriptlang.org/) ensures type safety and code maintainability.
-   **Containerization**: [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) are used for creating consistent development, production, and testing environments.
-   **Web Server**: [Nginx](https.www.nginx.com/) serves the production build.

## 3. Testing Strategy

The project employs a two-pronged testing approach:

-   **End-to-End (E2E) Testing**: [Nightwatch.js](https://nightwatchjs.org/) with [Selenium](https://www.selenium.dev/) runs tests against the production build in a Docker environment. This ensures that user flows and interactions work as expected from start to finish.
-   **Unit Testing**: [Vitest](https://vitest.dev/) is set up for unit testing components and business logic, although no unit tests have been written yet.

## 4. Development & Build Workflow

The entire development lifecycle is managed through Docker Compose, as defined in `docker-compose.yml`.

### Key Services:

-   `dev`: Runs the Astro development server with hot-reloading at `http://localhost:4321`.
-   `prod`: Serves the optimized, static production build via Nginx at `http://localhost:6543`.
-   `selenium`: Provides a standalone Chrome browser instance for Nightwatch to execute E2E tests.

### Important Commands:

-   **Start all services**:
    ```bash
    docker-compose up --build -d
    ```
-   **Run E2E tests**:
    ```bash
    npm run test:e2e
    ```
-   **Run unit tests**:
    ```bash
    npm run test:unit
    ```

## 5. Project Structure

-   `src/`: Contains all Astro source code, including pages, layouts, components, and data.
-   `public/`: Holds static assets like images and favicons.
-   `tests/`: Stores Nightwatch.js E2E test files.
-   `docker-compose.yml`: Defines and configures all project services.
-   `Dockerfile` & `Dockerfile.prod`: Specify the development and production container images, respectively.
-   `astro.config.mjs`: Astro-specific configuration.
-   `tailwind.config.mjs`: TailwindCSS configuration.
-   `nightwatch.conf.cjs`: Nightwatch.js test runner configuration.

## 6. Coding Conventions

-   **Formatting**: Code is formatted with [Prettier](https://prettier.io/).
-   **Linting**: [ESLint](https://eslint.org/) is configured to enforce code quality and catch errors early.
-   **TypeScript**: Strict type-checking is encouraged.
-   **Components**: Astro components are used to build the UI, promoting reusability and separation of concerns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelFisher1997)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MichaelFisher1997)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

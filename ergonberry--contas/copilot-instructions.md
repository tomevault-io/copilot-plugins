## contas

> This document outlines the conventions and guidelines for the Gemini AI assistant when working on this project.

# Gemini Guidelines

This document outlines the conventions and guidelines for the Gemini AI assistant when working on this project.

## Technologies

This project is built with the following technologies. Please adhere to their best practices and established patterns within the codebase.

- **React 18:** Use functional components with hooks.
- **TypeScript:** Adhere to the existing type definitions and add new ones as needed in the `src/types` directory.
- **Tailwind CSS:** Use Tailwind utility classes for styling. Do not write custom CSS unless absolutely necessary.
- **Chart.js:** Use Chart.js for creating interactive charts in the reports section.
- **Lucide React:** Use Lucide React for icons.
- **Docker:** Use Docker for containerization. The `docker-compose.yml` file is used for development and `docker-compose.prod.yml` for production.
- **Vite:** Vite is used as the bundler.

## Development Workflow

1.  **Run the application in development mode:**
    ```bash
    docker-compose up --build
    ```
2.  **Access the application at:** `http://localhost:5173`

## Testing

[TODO: Add instructions on how to run tests once a testing framework is set up.]

## Linting and Formatting

[TODO: Add instructions for linting and formatting once a linter is set up.]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ErgonBerry)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ErgonBerry)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

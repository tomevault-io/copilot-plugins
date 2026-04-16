## spec-kit-demo-expenses-tracker

> This is a Next.js project that serves as a demonstration for the "Spec-Kit" framework. Spec-Kit appears to be a sophisticated, AI-driven development workflow for ensuring structured, principle-based software engineering. It uses a combination of shell scripts, prompt templates, and Gemini CLI commands to guide the development process from specification to implementation.

# Gemini Project Context: Spec-Kit Demo

## Project Overview

This is a Next.js project that serves as a demonstration for the "Spec-Kit" framework. Spec-Kit appears to be a sophisticated, AI-driven development workflow for ensuring structured, principle-based software engineering. It uses a combination of shell scripts, prompt templates, and Gemini CLI commands to guide the development process from specification to implementation.

The core of the project is a standard Next.js application, but the main complexity and novelty lie in the `.specify`, `.gemini`, and `.github/prompts` directories, which define the Spec-Kit workflow.

**Key Technologies:**
- Next.js 16
- React 19
- TypeScript
- Tailwind CSS
- ESLint

## Building and Running

Standard Next.js commands are used to run and build the application.

- **Run development server:**
  ```bash
  npm run dev
  ```

- **Build for production:**
  ```bash
  npm run build
  ```

- **Start production server:**
  ```bash
  npm run start
  ```

- **Lint the code:**
  ```bash
  npm run lint
  ```

## Development Conventions

The project enforces a strict set of development principles defined in `.specify/memory/constitution.md`. This file acts as the source of truth for coding standards, testing practices, and architectural rules.

The Spec-Kit workflow is orchestrated through custom Gemini CLI commands defined in the `.gemini/commands/` directory (e.g., `speckit.plan.toml`). These commands execute scripts (located in `.specify/scripts/`) that use detailed prompts (from `.github/prompts/`) to perform tasks such as:

- Creating implementation plans.
- Generating design artifacts like data models and API contracts.
- Performing research tasks to resolve technical uncertainties.
- Ensuring all development activities align with the project's constitution.

Any new development should likely be initiated through the `speckit.*` commands rather than by manually editing files, to ensure compliance with the established workflow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fsheee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

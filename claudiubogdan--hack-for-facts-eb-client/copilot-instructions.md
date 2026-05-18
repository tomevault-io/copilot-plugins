## hack-for-facts-eb-client

> This is a web application for analyzing public spending data. It is designed for the public sector and journalists to easily find anomalies in public spending. The application provides features for anomaly detection and data exploration through an intuitive interface. A key feature is the AI Filter Generator, which allows users to generate filters using natural language queries.

# Project Overview

This is a web application for analyzing public spending data. It is designed for the public sector and journalists to easily find anomalies in public spending. The application provides features for anomaly detection and data exploration through an intuitive interface. A key feature is the AI Filter Generator, which allows users to generate filters using natural language queries.

**Key Technologies:**

* **Frontend:** React.js, Tailwind CSS, TypeScript, Shadcn UI
* **Data Fetching:** Tanstack Query, Axios
* **Routing:** Tanstack Router
* **State Management:** Zustand
* **Build Tool:** Vite
* **Testing:** Manual testing

# Building and Running

**Installation:**

```bash
yarn install
```

**Type checking:**

```bash
npx tsc -b --noEmit
```

**Running tests:**

No tests yet. Only manual testing.

# Instructions for Git Operations in This Repository

## Stash Usage Rules

* Before making any modifications via shell commands, always run `git stash push --keep-index -m "gemini-temp-[current-date-time]-[summary-of-changes]"` to save the current working state without losing staged changes. Replace [current-date-time] with the actual timestamp (e.g., "gemini-temp-2025-08-07-10AM").

* Use this special "gemini-temp-" tag for all stashes you create to mark them as AI-generated temporaries.
* Always check `git stash list` first to avoid overwriting existing stashes.
* Never use stash commands that could revert or lose state, like generic `git stash` without --keep-index.

## Example Workflow

1. Stash: `git stash push --keep-index -m "gemini-temp-2025-08-07-10AM-fix-bug-in-map-view"`
2. Perform safe modifications (e.g., via allowed Git commands).
3. Retrieve: `git stash apply` (if needed).

# Development Conventions

* The project uses TypeScript for static typing.
* ESLint is used for linting. You can run the linter with `yarn lint`.
* The project follows the conventional commits specification for commit messages.
* The application uses Tanstack Router for routing. Route definitions are generated automatically. To regenerate the routes, run `yarn router:generate`.
* For type checking, run `npx tsc -b --noEmit`.

# Sentry Integration Spec

* Goals
  * Capture runtime errors and React render errors in production via Sentry
  * Enable performance tracing (configurable sampling) and TanStack Router navigation spans
  * Respect cookie consent: do not send any events without analytics consent
  * Provide end-user feedback via Sentry's Feedback widget, with programmatic open
  * Upload sourcemaps in CI/build using `@sentry/vite-plugin`

# Context

Use the files in the `gpt-prompt` directory to help you understand the project.

---
> Source: [ClaudiuBogdan/hack-for-facts-eb-client](https://github.com/ClaudiuBogdan/hack-for-facts-eb-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

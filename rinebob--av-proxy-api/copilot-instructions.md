## av-proxy-api

> * Always read all markdown files in the `planning` directory at the start of a new conversation to understand the project's architecture, goals, style, and constraints.


# Project Awareness & Context

* Always read all markdown files in the `planning` directory at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
* Check `TASK.md` before starting a new task. If the task isn’t listed, add it with a brief description and today's date. If that file doesn’t exist prompt me to create one.
* Use consistent naming conventions, file structure, and architecture patterns as described in `PLANNING.md`.
* When in **Chat Mode** always make code suggestions as ‘Proposed Changes’ in a proposed changes window with the ‘apply’ button available.

---

# Code Structure & Modularity

* Never create a file longer than 500 lines of code. If a file approaches this limit, refactor by splitting it into subcomponents or helper files.
* Organize code into clearly separated directories, grouped by feature or responsibility.
* Use clear, consistent imports (prefer relative imports within packages).

---

# Tech Stack

* Please use the **Brave API** to look up and research documentation for all frameworks/libraries.
* Use **Angular** and **TypeScript** as the framework and language. Don’t ever use React or Vue or Svelte.
* Use **NgRx Signal Store** for state management. When scaffolding the signal store, always use the NgRx Signal Store syntax, not Angular service.
* Use **RxJs** wherever needed.
* Don’t use promises.
* Deployment will likely be on **Firebase**.
* Any backend database/compute requirements will likely be **Firestore/Cloud Functions**.

---

# Angular Specific Requirements

* When you scaffold components, just generate code similar to what the Angular CLI would generate. Don't generate a completed component during the scaffold process. I just want the empty component.
* Always use the **latest Angular features and syntax**. Don't use features or syntax that has been superseded.
* Always use the `inject` function for DI.
* Always use new **control flow syntax** (`@if{}`, `@for{}` etc) instead of `*ngIf` or `*ngFor`.
* Always new **signals** and the variants `signal input/output`, `linkedSignals`, `model` etc.
* Always use **standalone components**. Never use `ng modules`.
* Always use Angular’s new **self-closing tags** in HTML templates.
* Always use separate files for template and styles. Don't inline.
* Never use getters in HTML templates. Always use Angular signals.
* In Angular `component.ts` files, always put imports, constants, interfaces etc above the `@Component` decorator, never below it.
* Follow **Angular and TypeScript style guides**, use type hints.
* Import types whenever available.
* Format with a modern TypeScript formatter.
* Use `npx` to run Angular CLI commands.

---

# Style & Conventions

* Always use **flexbox** for layouts. Use CSS Grid sparingly.
* Always use **Sass** for styling.
* Always put styles in the `.scss` file. Never use inline styles.
* Don’t use `ngStyle` or `ngClass`.
* For the theme CSS, always use **variables** and never hard-code values for any numeric property.
* Always use `rem` or `em` for sizes and dimensions. Set the default value as `1rem = 16px`.
* Always use the `rem()` function instead of hard-coding values.
* Pixel values can be used for styling elements like drop shadows.
* Always use **Sass mixins** when styling components.
* Always use **Sass variables** when possible in mixins. Define the variables before they are used.
* If a `_theme.scss` file is not present, use global `_mixins.scss` and `_variables.scss` files and `@use` these in component style sheets.
* In the terminal, if a process is running on port 4200 when you try to restart the app, automatically kill that process and try again.
* Always create **light and dark mode themes**.

---

# Testing & Reliability

## End-to-End Testing

* Use **Cypress** or **Playwright** to create end-to-end tests for all user success journeys.

## Unit Testing

* Always use **Jest** to create unit tests. Do not use Jasmine/Karma (deprecated).
* Create full unit tests for all new features (functions, classes, routes, etc).
* After updating any logic, check whether existing unit tests need to be updated. If so, do it.
* Tests should live in a `/tests` folder mirroring the main app structure.
* Include at least:
    * 1 test for expected use
    * 1 edge case
    * 1 failure case

---

# Task Completion

* Mark completed tasks in `TASK.md` immediately after finishing them.
* Add new sub-tasks or TODOs discovered during development to `TASK.md` under a “Discovered During Work” section.

---

# Documentation & Explainability

* Write **JSDoc comments** for all classes and functions.
* Update `README.md` when new features are added, dependencies change, or setup steps are modified.
* Comment non-obvious code and ensure everything is understandable to a mid-level developer.
* When writing complex logic, add an inline `# Reason:` comment explaining the why, not just the what.

---

# AI Behavior Rules

* Never assume missing context. Ask questions if uncertain.
* Never hallucinate libraries or functions – only use known, verified Angular/Typescript and related packages.
* Never use React or refer to React.
* Always confirm file paths and component names exist before referencing them in code or tests.
* Never delete or overwrite existing code unless explicitly instructed to or if part of a task from `TASK.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rinebob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->

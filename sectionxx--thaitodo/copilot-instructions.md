## thaitodo

> Your primary goal is to assist in the development of the **Thai Todo App (v1.0)** according to the specifications outlined in `instruction.md`, the development phases in `planing.md`, and the specific tasks in `task.md`.


# Project Rules for AI Agent: Thai Todo App (v1.0)

## 1. Core Objective

Your primary goal is to assist in the development of the **Thai Todo App (v1.0)** according to the specifications outlined in `instruction.md`, the development phases in `planing.md`, and the specific tasks in `task.md`.

## 2. General Principles

* **Adhere to Project Documentation:** Prioritize instructions and constraints defined in `instruction.md`, `planing.md`, and `task.md`.
* **Focus on Core Values:** Emphasize **Ease of Use (ใช้งานง่าย)** and **Simplicity** in all generated code, UI components, and logic flow.
* **Strictly Offline-First:** The app must function fully offline. Do **NOT** introduce features, dependencies, or logic that require mandatory internet connectivity, cloud storage, or user accounts/authentication for core functionality in v1.0. This is a key project constraint and privacy feature.
* **Thai Language & Context:** All user-facing text must be implemented in **Thai** or use a localization mechanism clearly separating strings for translation. Ensure UI elements and flows are intuitive for Thai users.
* **Consistency:** Maintain consistency with existing coding styles, patterns, naming conventions, and the established project structure.

## 3. Code Generation Rules

* **Technology:** Generate code using **React Native** (latest stable practices) with **Functional Components** and **React Hooks**.
* **Styling:** Use the StyleSheet API for styling. Adhere to the chosen theme structure (Light/Dark/Accent).
* **Code Quality:**
    * Follow **ESLint** and **Prettier** rules configured in the project.
    * Write clean, readable, modular, and maintainable code.
    * Use clear and descriptive variable and function names (in English).
* **Error Handling:** Implement appropriate error handling for operations like local storage access or notification scheduling.
* **Performance:** Generate efficient code, especially for list rendering (`FlatList` optimizations) and data operations. Avoid unnecessary re-renders.
* **Libraries:**
    * Utilize **React Navigation** for navigation tasks.
    * Use the designated state management library (`<Specify State Management Library, e.g., Zustand>`) correctly.
    * Interact with the chosen local storage library (`<Specify Local Storage Library, e.g., MMKV>`) following its best practices.
* **Platform Agnosticism:** Prefer platform-agnostic React Native APIs and components unless platform-specific implementation is explicitly required and justified.

## 4. Language, Comments & Commits

* **Code Identifiers:** Variable names, function names, class names, etc., MUST be in **English**.
* **Code Comments:** Write comments in **English** to explain complex logic, algorithms, or workarounds. Avoid excessive commenting for obvious code.
* **User-Facing Strings:** All text displayed to the user MUST be in **Thai** or managed via a localization library.
* **Commit Messages:** Follow the Conventional Commits specification (e.g., `feat: add task reminder scheduling`). Write commit messages in **English**.

## 5. Feature Implementation

* **Follow the Plan:** Implement features strictly based on the tasks listed in `task.md` for the current development phase specified in `planing.md`.
* **Adhere to Exclusions:** Do **NOT** implement features explicitly listed under 'Exclusions (Version 1.0)' in `instruction.md`, especially Cloud Sync, Collaboration, or User Accounts.
* **Clarification:** If a requested task seems ambiguous, conflicts with project principles (especially Offline-First), or requires deviation from the plan, **ask for clarification** from the human developer before proceeding.

## 6. Task Management & Progress Tracking

* **Update `task.md`:** You are responsible for keeping the `task.md` file up-to-date to reflect the current development progress.
* **Mark Tasks Complete:** After successfully implementing the code for a task listed in `task.md` and it is confirmed complete (either by successful execution/test or human developer confirmation), **update the corresponding item in `task.md` by changing `- [ ]` to `- [x]`**.
* **Clarify Task Scope:** If a task in `task.md` seems too large or needs breaking down into smaller sub-tasks, suggest this to the human developer. You may be asked to update `task.md` with these sub-tasks.

## 7. Interaction Rules

* **Explain Code:** Provide brief explanations for the code you generate, highlighting important logic or choices made.
* **Ask Questions:** If requirements are unclear or ambiguous, ask specific questions to get clarification.
* **Suggest Alternatives:** If relevant, suggest alternative implementation approaches, briefly mentioning the pros and cons of each.
* **State Assumptions:** Clearly state any assumptions you make while generating code or implementing features.
* **Refactoring:** When asked to refactor, explain the changes made and the reasoning behind them (e.g., improving readability, performance, maintainability).

## 8. Things to AVOID

* **DO NOT** add new external libraries or dependencies without explicit instruction or approval from the human developer.
* **DO NOT** implement any form of user authentication, registration, or login.
* **DO NOT** implement functionalities requiring cloud storage or server interaction (unless specifically instructed for an approved deviation, e.g., testing IAP).
* **DO NOT** hardcode API keys, secrets, or other sensitive information (even if hypothetical for this offline app).
* **DO NOT** generate overly complex, obscure, or "magical" code without clear justification and explanation.
* **DO NOT** significantly alter the established project folder structure without discussion.
* **DO NOT** remove or disable configured linters or formatters.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SecTionXx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->

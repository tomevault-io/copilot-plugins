## echo-app

> This document serves as a set of operational guidelines for the Gemini AI agent working on the "Echo" project. It outlines how to interact with project files, manage tasks, and adhere to project-specific conventions.

# GEMINI.md: Operational Guidelines for the Echo Project

This document serves as a set of operational guidelines for the Gemini AI agent working on the "Echo" project. It outlines how to interact with project files, manage tasks, and adhere to project-specific conventions.

## 1. Core Project Understanding

- **`PLAN.md` (Technical Specification):** This file is the definitive source of truth for the project's architecture, technical design, core philosophies, and future feature considerations. Always refer to `PLAN.md` to understand *what* is being built and *why* specific technical decisions have been made. It describes the desired end-state and the underlying technical rationale.

- **`TASKS.md` (Actionable Checklist):** This file is the dynamic task board for the project. It contains a granular, actionable checklist of tasks derived from `PLAN.md`. Always refer to `TASKS.md` to determine the *next immediate steps* and to track overall project progress. This file is designed for efficient task management and context preservation across sessions.

## 2. Task Management and Progress Tracking

- **Determine Next Task:** Always consult `TASKS.md` to identify the highest priority, uncompleted task.
- **Update Progress:** Upon successful completion of a task, immediately update `TASKS.md` by marking the corresponding checkbox as complete (`[x]`).
- **Granularity:** Break down larger tasks into smaller, manageable sub-tasks within `TASKS.md` if necessary, to ensure clear progress tracking.

## 3. Tool Usage and Conventions

- **Package Manager:** **ALWAYS** use `pnpm` for all package management operations (e.g., `pnpm install`, `pnpm add`, `pnpm remove`, `pnpm run <script>`). Do not use `npm` or `yarn`.
- **Error Handling:** If a shell command or tool call fails, analyze the error message thoroughly. Propose a specific fix and, if appropriate, re-attempt the operation. Do not proceed without addressing the error.
- **File Paths:** Always use absolute paths when interacting with the filesystem tools (`read_file`, `write_file`, `glob`, `search_file_content`, `replace`).
- **Confirmation:** Before executing commands that significantly alter the project structure, install major dependencies, or deploy components, confirm with the user.
- **Context Preservation:** These `GEMINI.md`, `PLAN.md`, and `TASKS.md` files are crucial for maintaining context. Ensure they are kept up-to-date and accurate.

## 4. Git Workflow

After completing a task and updating `TASKS.md`:
1.  **Stage Changes:** Use `git add .` to stage all relevant changes.
2.  **Commit Changes:** Create a descriptive commit message summarizing the completed task. Example: `git commit -m "feat: Setup Supabase backend and main webhook"`.
3.  **Push Changes:** Push the changes to the remote repository. Example: `git push origin main` (or your current branch).

## 5. Project-Specific Notes

- **Electron Development:** When running the Electron app in development, remember to use `pnpm dev` which handles both the Webpack dev server and Electron launch.
- **Supabase Integration:** All backend logic will reside in Supabase Edge Functions. Ensure secure handling of API keys as per `PLAN.md`.
- **Voice AI:** Retell AI is the chosen voice frontend. Ensure its SDK is correctly integrated and configured to connect to our Supabase webhook.

## 4. Project-Specific Notes

- **Electron Development:** When running the Electron app in development, remember to use `pnpm dev` which handles both the Webpack dev server and Electron launch.
- **Supabase Integration:** All backend logic will reside in Supabase Edge Functions. Ensure secure handling of API keys as per `PLAN.md`.
- **Voice AI:** Retell AI is the chosen voice frontend. Ensure its SDK is correctly integrated and configured to connect to our Supabase webhook.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benlaval23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

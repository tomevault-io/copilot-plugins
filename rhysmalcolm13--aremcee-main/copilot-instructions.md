## aremcee-main

> ALWAYS USE WHEN developing new features, fixing bugs, or refactoring code.


- **1. Understand Requirements & Task Definition:**
  - Clearly define the requirements (e.g., from a PRD, issue tracker, or user request).
  - Identify the core goal (τ) and affected areas of the codebase.
  - **Read Related Rules:** Before starting, read *all* relevant `.cursor/rules/*.mdc` files (e.g., `nextjs.mdc`, `react.mdc`, `drizzle.mdc`, `auth.mdc` if touching those areas).
  - Consult the [project-index.mdc](mdc:.cursor/rules/project-index.mdc) (once created) to understand file relationships.
  - **Create/Update Task:** Based on the requirements, create a new task or update an existing one in [tasks.json](mdc:tasks/tasks.json) and regenerate [tasks.md](mdc:tasks/tasks.md). 
    - **Follow [tasks.mdc](mdc:.cursor/rules/tasks.mdc) strictly** for schema, ID generation (find max ID + 1), status ('pending'), priority, dependencies, description, and potential subtask decomposition.
    - Example: If implementing a user request for profile editing, create a task like `{"id": <next_id>, "title": "Implement user profile editing", "status": "pending", ...}`.

- **2. Branching Strategy:**
  - Create a new feature branch from the main development branch (e.g., `main` or `develop`).
  - Use a descriptive branch name, potentially including the task ID (e.g., `feat/task-5-user-profile`, `fix/task-12-login-redirect`).

- **3. Implementation (Ω₄):**
  - **Update Task Status:** Before starting implementation, update the corresponding task's status to 'in_progress' in [tasks.json](mdc:tasks/tasks.json) and regenerate [tasks.md](mdc:tasks/tasks.md). Follow [tasks.mdc](mdc:.cursor/rules/tasks.mdc).
  - **Follow Rules:** Strictly adhere to the guidelines defined in the relevant `.cursor/rules/*.mdc` files read in Step 1.
  - **Code Structure:** Maintain the existing project structure ([directory-structure.mdc](mdc:.cursor/rules/directory-structure.mdc)). Place new files in appropriate locations.
  - **Modularity:** Write modular, reusable code. Extract complex logic into utility functions or services (e.g., in `lib/` subdirectories).
  - **Typing:** Use TypeScript effectively ([typescript.mdc](mdc:.cursor/rules/typescript.mdc)).
  - **UI:** Use Shadcn components ([shadcn-ui.mdc](mdc:.cursor/rules/shadcn-ui.mdc)) and Tailwind CSS ([tailwind.mdc](mdc:.cursor/rules/tailwind.mdc)) for UI development.
  - **Database:** Follow Drizzle conventions ([drizzle.mdc](mdc:.cursor/rules/drizzle.mdc)) for schema changes (inform user) and queries.
  - **Authentication:** Adhere to authentication patterns ([auth.mdc](mdc:.cursor/rules/auth.mdc)).
  - **API Routes:** Follow API route conventions ([api-routes.mdc](mdc:.cursor/rules/api-routes.mdc)).

- **4. File Headers & Indexing:**
  - **Universal Header:** If implementing the Universal Header standard, ensure every new or modified file includes it and keeps it updated.
  - **Project Index:** **Concurrently** update the [project-index.mdc](mdc:.cursor/rules/project-index.mdc) with any new files added or significant changes made to existing ones (purpose, dependencies).

- **5. Testing (Ω₅ - Manual/User):**
  - Test the changes thoroughly locally (`pnpm dev`).
  - Verify functionality across different scenarios and edge cases.
  - (If applicable) Write unit or integration tests (no testing framework currently detected).

- **6. Linting & Formatting:**
  - Run linters and formatters (e.g., `eslint`, `prettier` via `package.json` scripts) and fix any reported issues.
  - Address TypeScript compiler errors.

- **7. Commit & Push:**
  - Commit changes with clear, descriptive messages, referencing the task ID (e.g., `feat(profile): Add edit form (#5)`).
  - Push the feature branch to the remote repository.

- **8. Pull Request & Review (Ω₅):**
  - Create a Pull Request (PR) targeting the main development branch.
  - Describe the changes clearly in the PR description, referencing the task ID.
  - Link to the relevant task in [tasks.md](mdc:tasks/tasks.md).
  - Request reviews from team members.
  - Address feedback and make necessary revisions.

- **9. Merge & Cleanup:**
  - Once approved, merge the PR.
  - **Update Task Status:** Update the corresponding task's status to 'completed' in [tasks.json](mdc:tasks/tasks.json) and regenerate [tasks.md](mdc:tasks/tasks.md). Follow [tasks.mdc](mdc:.cursor/rules/tasks.mdc).
  - Delete the feature branch after merging.

- **Schema Migrations:**
  - If database schema changes were made ([lib/db/schema.ts](mdc:lib/db/schema.ts)), **notify the user** that they need to generate and apply migrations using Drizzle Kit commands (e.g., `pnpm drizzle-kit generate`, `pnpm drizzle-kit push`). Do not run these commands yourself.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rhysmalcolm13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

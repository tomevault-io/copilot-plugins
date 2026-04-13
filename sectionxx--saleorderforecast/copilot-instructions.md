## saleorderforecast

> These rules are designed to ensure that any modifications made by the AI Agent align with the project's existing architecture, standards, and goals.




## Project Rules and Guidelines for AI Agent (SaleOrderForecast Project)

These rules are designed to ensure that any modifications made by the AI Agent align with the project's existing architecture, standards, and goals.

1.  **Understand Existing Codebase First:**
    * Before making changes, thoroughly analyze the relevant parts of the existing codebase (structure, patterns, and dependencies).
    * Refer to existing documentation (`README.md`, `PROJECT_PLAN.md`, `TASK_BREAKDOWN.md`, `UI_AND_TESTING_PLAN.md`) for context and intent.

2.  **Adhere to Technology Stack:**
    * **Frontend:** Continue using HTML, CSS (including TailwindCSS and the existing modular CSS structure: `css/variables.css`, `css/layout.css`, `css/components.css`, etc.), and JavaScript.
    * **JavaScript:** Write modern ES6+ JavaScript. Utilize existing libraries like Chart.js and Feather Icons appropriately.
    * **Backend:** For local development, align with the Node.js/Express setup (`server.js`). For production API endpoints, ensure changes are compatible with Vercel Serverless Functions (`api/` directory, `vercel.json`).
    * **Data Source:** Interactions with Google Sheets API should be robust and efficient.

3.  **Maintain Code Style and Quality:**
    * Follow the existing code style and formatting. If an ESLint configuration exists (`.eslintrc.js`), adhere strictly to its rules.
    * Prioritize code readability, maintainability, and clarity.
    * Write clear, concise, and meaningful comments where necessary, especially for complex logic or non-obvious decisions.

4.  **Emphasize Modularity and Reusability:**
    * Break down complex functions and components into smaller, single-responsibility modules/units, following the established modular structure (e.g., within `js/components/`, `js/services/`, `js/utils/`).
    * Strive to create reusable functions and UI components to reduce code duplication.
    * When implementing ES6 modules, ensure clear exports and imports.

5.  **Preserve and Enhance Functionality:**
    * The primary goal of refactoring is to improve the codebase, not to introduce new features unless specified in the prompt.
    * Ensure all existing functionalities remain intact and perform as expected after refactoring.
    * If performance improvements are part of the task, demonstrate or explain the impact.

6.  **Follow Security Best Practices:**
    * For authentication and authorization tasks (e.g., JWT implementation, role-based access), implement standard security practices.
    * Sanitize inputs and validate data on both client and server sides to prevent common vulnerabilities.
    * Be mindful of how environment variables and sensitive data are handled (refer to `.env` usage and Vercel environment configuration).

7.  **Respect Directory Structure:**
    * Maintain the existing project directory structure.
    * If new files or directories are created, they should logically fit within the established organization (e.g., new UI components in `js/components/`, new utility functions in `js/utils/`).

8.  **Manage Dependencies Carefully:**
    * Avoid introducing new external libraries or dependencies unless absolutely necessary and justified.
    * If a new dependency is added, ensure it's a well-maintained and reputable library, and update `package.json` accordingly.

9.  **API Design and Integrity:**
    * When refactoring the API layer (`api/getSheetData.js`, `auth/` services), ensure consistency in request/response formats.
    * Implement proper HTTP status codes and error responses.
    * API endpoints should remain secure and performant.

10. **Testing Considerations:**
    * Ensure that refactored code does not break existing tests (refer to files in `tests/unit/`).
    * While the refactoring prompt doesn't explicitly demand writing new tests, any significantly refactored or new critical logic should ideally be testable. If the AI can suggest or create unit tests for new modules, that would be beneficial.

11. **Iterative Changes and Version Control (Conceptual):**
    * While the AI won't directly use Git, it should make changes in a way that is conceptually easy to review and integrate (e.g., addressing one task or module at a time).

12. **Focus on the Provided Prompt:**
    * Prioritize addressing the specific tasks and goals outlined in the refactoring prompt. Avoid going out of scope unless it's a direct and necessary consequence of a requested change.

13. **Clear Communication of Changes:**
    * When providing the refactored code, clearly summarize the changes made, the rationale behind them, and any potential impacts or further considerations.

By following these rules, the AI Agent can contribute effectively to improving the SaleOrderForecast project in a structured and beneficial way.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SecTionXx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->

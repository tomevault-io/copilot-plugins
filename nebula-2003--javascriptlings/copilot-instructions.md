## javascriptlings

> JavaScriptlings is an interactive learning tool designed to help users learn basic JavaScript concepts by solving small, self-contained exercises. Inspired by Rustlings, it leverages existing JavaScript task files and their corresponding Vitest test files to guide users through the learning process.

# JavaScriptlings

JavaScriptlings is an interactive learning tool designed to help users learn basic JavaScript concepts by solving small, self-contained exercises. Inspired by Rustlings, it leverages existing JavaScript task files and their corresponding Vitest test files to guide users through the learning process.

## How it Works

Each exercise consists of a JavaScript file in the `tasks/` directory and a test file in the `tests/` directory. Users are expected to modify the JavaScript task files to solve the problems. The `javascriptlings` CLI tool then runs the tests to verify the solutions.

## Getting Started

To run JavaScriptlings and check your progress, use the following command:

```bash
npm run javascriptlings
```

This command will:
1.  Discover all test files in the `tests/` directory.
2.  Determine the status of each exercise:
    *   **SKIPPED:** If the main `describe` or `it` blocks within the test file are commented out. The tool will provide a hint on how to enable the exercise.
    *   **ATTEMPTED:** If the test blocks are uncommented. The tool will then run the corresponding tests.
3.  Report the results for each exercise as `PASS`, `FAIL`, or `SKIPPED`.

## Solving Exercises

1.  **Navigate to a Test File:** Go to a test file in the `tests/` directory (e.g., `tests/basics/1.even-numbers.test.js`).
2.  **Uncomment the Test Block(s):** Uncomment the `describe` or `it` blocks that are currently commented out. This signals to the `javascriptlings` tool that you are ready to attempt this exercise.
3.  **Implement the Solution:** Open the corresponding JavaScript task file in the `tasks/` directory (e.g., `tasks/1.basics/1.even-numbers.js`) and write your solution.
4.  **Run JavaScriptlings:** Execute `npm run javascriptlings` again to check if your solution passes the tests.
5.  **Iterate:** If the tests fail, refine your solution and re-run the tool until all tests pass.

## Project Structure

*   `tasks/`: Contains the JavaScript files for each exercise that users need to complete.
*   `tests/`: Contains the Vitest test files for each exercise. These files are used by the `javascriptlings` tool to verify solutions.
*   `src/runner.js`: The core CLI tool that orchestrates the exercise running and reporting.

## Coding Standards

To maintain consistency and readability across the project, please adhere to the following coding standards:

*   **Language:** JavaScript (ES Modules).
*   **Formatting:** Follow the existing formatting conventions within the `tasks/` and `tests/` directories. Use consistent indentation (2 spaces preferred if not already established), spacing, and line breaks.
*   **Naming Conventions:**
    *   **Variables and Functions:** Use `camelCase` (e.g., `myVariable`, `calculateSum`).
    *   **Constants:** Use `UPPER_SNAKE_CASE` for global constants (e.g., `MAX_VALUE`).
    *   **File Names:** Use `kebab-case` for file names (e.g., `even-numbers.js`).
*   **Imports/Exports:** Use ES Module syntax (`import ... from '...'` and `export ...`).
*   **Comments:** Use comments to explain complex logic or non-obvious parts of the code. Avoid excessive commenting for self-explanatory code.
*   **Testing:** All tests are written using `vitest`. Ensure new tests follow the existing `vitest` patterns (`describe`, `it`, `expect`).
*   **Error Handling:** Implement appropriate error handling where necessary, especially for functions that might encounter unexpected input or external issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nebula-2003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nebula-2003)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

## listall

> - **Code must always build successfully.** This is non-negotiable.


## Behavioral Rules

### Build Validation (CRITICAL)
- **Code must always build successfully.** This is non-negotiable.
- After ANY code changes, run the appropriate build command to verify compilation.
- If build fails, immediately use the `<fix>` workflow to resolve errors.
- Never leave the project in a broken state.
- For iOS/macOS projects: Use `xcodebuild` commands
- For web projects: Use `npm run build` or equivalent
- Document any persistent build issues in `docs/learnings.md`

### Testing Integration (CRITICAL)
- **Every task must end with successful build AND 100% successful tests.** This is non-negotiable.
- **ALL TESTS MUST ACTUALLY PASS** - Never consider a task complete with failing tests.
- Write tests as you implement features, not as a separate phase.
- After ANY code changes, run tests to verify functionality works correctly.
- If tests fail, immediately fix the issues before considering the task complete.
- **MANDATORY**: Run actual test commands and verify 100% pass rate before task completion.
- For iOS/macOS projects: Use `xcodebuild test` commands and verify "Test Succeeded" output
- For web projects: Use `npm test` or equivalent and verify all tests pass
- Prefer fast test execution during development (skip performance tests, run unit tests only)
- **NO EXCEPTIONS**: Tasks are incomplete until tests actually pass, not just expected to pass
- Document any persistent test issues in `docs/learnings.md`

### Documentation (CRITICAL)
- **Every completed task must be recorded in ai_changelog.md.** This is non-negotiable.
- After completing ANY significant work, update the changelog with comprehensive details.
- Include technical details, files modified, build status, and next steps.
- Document any persistent issues or learnings in appropriate files.
- Never skip the recording step - it's as critical as build validation.

### Code Quality Standards
- Follow the established architecture patterns in the project
- Ensure proper imports and dependencies
- Use consistent naming conventions
- Add appropriate error handling
- Write clean, readable code

## Request Processing Steps
Follow these steps sequentially for **every** user request and follow behavioral rules above:

1.  **Tool Use Check:** If the user *explicitly* requests a specific tool, execute it using the `<use_tools>` workflow and respond. **Stop processing here.**
2.  **Initial Analysis:**
    * Read the user request carefully.
    * Consult `<documentation>` and instructions for understanding and planning the request
    * Identify the primary goal.
    * Determine the most relevant `<workflow>`. Default to `<research>` if unclear.
    * List affected architecture areas / subsystems. 
    * **Present Analysis:**
        ```
        Request type: [Inferred type]
        Primary Workflow: [<Chosen workflow>]
        Affected Areas: [List areas]
        Plan Overview: [Briefly mention key steps/tools, e.g., "Read docs, formulate prompt for edit_file, run tests"]
        ```
3.  **Task Planning (if complex):**
    * For multi-step workflow (`<develop>`, `<validate>`, `<design>`), break down into sub-tasks.
    * Use `edit_file` to add sub-tasks to `todo.md` (marked ❌). Include development, testing, documentation steps.
    * Inform the user the plan is in `todo.md`.
    * **IMPORTANT:** Complete ONE logical feature/task at a time, then stop for user review and commit.
4.  **Workflow Execution:**
    * State: `Starting workflow: <workflow name>...`
    * Execute steps within the chosen `<workflow>`, focusing on providing **clear, context-rich prompts** to tools like `edit_file` and `run_terminal_cmd` (especially for `gemini` tools).
    * Adhere strictly to `<behavior_rules>`.
    * **MANDATORY BUILD VALIDATION:** After any code changes, execute `<build_validate>` workflow to ensure the project builds successfully.
    * **MANDATORY TEST VALIDATION:** After any code changes, run actual test commands and verify 100% pass rate.
    * **CRITICAL**: Tests must actually pass - use `xcodebuild test` or equivalent and verify success output.
    * Write tests for new features as you implement them.
    * Review outputs and iterate with follow-up prompts if necessary.
    * **STOP after completing ONE logical feature/task** for user review and commit.
    * State: `Completed Workflow: <workflow name>. Ready for review and commit.`
    * **REMINDER**: Don't forget final recording step if this completes the entire request.
5.  **Final Recording (MANDATORY on task completion):**
    * Once the *entire* request is fulfilled:
        * Summarize significant changes.
        * **MANDATORY**: Execute `<record>` workflow (update `ai_changelog.md`, `todo.md`).
        * Ensure `learnings.md` was updated during `<fix>` or `<invent>` workflows if applicable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chmc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

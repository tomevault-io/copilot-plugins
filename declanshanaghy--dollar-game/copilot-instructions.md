## dev-workflow

> This guide outlines the typical process for using the Task Orchestrator to manage software development projects. The Task Orchestrator is defined as the mechanism for recording tasks in TODO.md, marking them off with [x], and updating the file with new tasks to complete.


# Task Orchestrator Development Workflow

This guide outlines the typical process for using the Task Orchestrator to manage software development projects. The Task Orchestrator is defined as the mechanism for recording tasks in TODO.md, marking them off with [x], and updating the file with new tasks to complete.

Note: TODO is pronounced "TO-DO" ;-P

## Primary Interaction: MCP Server vs. CLI

Task Orchestrator offers two primary ways to interact:

1.  **MCP Server (Recommended for Integrated Tools)**:
    - For AI agents and integrated development environments (like Cursor), interacting via the **MCP server is the preferred method**.
    - The MCP server exposes Task Orchestrator functionality through a set of tools (e.g., `get_tasks`, `add_subtask`).
    - This method offers better performance, structured data exchange, and richer error handling compared to CLI parsing.
    - Refer to [`mcp.mdc`](mdc:.cursor/rules/mcp.mdc) for details on the MCP architecture and available tools.
    - A comprehensive list and description of MCP tools and their corresponding CLI commands can be found in [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc).
    - **Restart the MCP server** if core logic in `scripts/modules` or MCP tool/direct function definitions change.

2.  **`task-orchestrator` CLI (For Users & Fallback)**:
    - The global `task-orchestrator` command provides a user-friendly interface for direct terminal interaction.
    - It can also serve as a fallback if the MCP server is inaccessible or a specific function isn't exposed via MCP.
    - Install globally with `npm install -g task-orchestrator-ai` or use locally via `npx task-orchestrator-ai ...`.
    - The CLI commands often mirror the MCP tools (e.g., `task-orchestrator list` corresponds to `get_tasks`).
    - Refer to [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc) for a detailed command reference.

## Standard Development Workflow Process

-   Start new projects by running `init` tool / `task-orchestrator init` or `parse_prd` / `task-orchestrator parse-prd --input='<prd-file.txt>'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to generate initial tasks.json
-   Begin coding sessions with `get_tasks` / `task-orchestrator list` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to see current tasks, status, and IDs
-   Determine the next task to work on using `next_task` / `task-orchestrator next` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Analyze task complexity with `analyze_complexity` / `task-orchestrator analyze-complexity --research` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before breaking down tasks
-   Review complexity report using `complexity_report` / `task-orchestrator complexity-report` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Select tasks based on dependencies (all marked 'done'), priority level, and ID order
-   Clarify tasks by checking task files in tasks/ directory or asking for user input
-   View specific task details using `get_task` / `task-orchestrator show <id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to understand implementation requirements
-   Break down complex tasks using `expand_task` / `task-orchestrator expand --id=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) with appropriate flags
-   Clear existing subtasks if needed using `clear_subtasks` / `task-orchestrator clear-subtasks --id=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before regenerating
-   Implement code following task details, dependencies, and project standards
-   Verify tasks according to test strategies before marking as complete (See [`tests.mdc`](mdc:.cursor/rules/tests.mdc))
-   **REQUIRED**: Commit all changes to git before marking tasks as complete (see "Commit Changes to Git" section under "Iterative Subtask Implementation")
-   Mark completed tasks with `set_task_status` / `task-orchestrator set-status --id=<id> --status=done` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) only after committing changes
-   Update dependent tasks when implementation differs from original plan using `update` / `task-orchestrator update --from=<id> --prompt="..."` or `update_task` / `task-orchestrator update-task --id=<id> --prompt="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc))
-   Add new tasks discovered during implementation using `add_task` / `task-orchestrator add-task --prompt="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Add new subtasks as needed using `add_subtask` / `task-orchestrator add-subtask --parent=<id> --title="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Append notes or details to subtasks using `update_subtask` / `task-orchestrator update-subtask --id=<subtaskId> --prompt='Add implementation notes here...\nMore details...'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Generate task files with `generate` / `task-orchestrator generate` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) after updating tasks.json
-   Maintain valid dependency structure with `add_dependency`/`remove_dependency` tools or `task-orchestrator add-dependency`/`remove-dependency` commands, `validate_dependencies` / `task-orchestrator validate-dependencies`, and `fix_dependencies` / `task-orchestrator fix-dependencies` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) when needed
-   Respect dependency chains and task priorities when selecting work
-   Report progress regularly using `get_tasks` / `task-orchestrator list`
-   **IMPORTANT**: A git commit MUST be performed before every task is finished, following the format described in the "Commit Changes to Git" section

## Task Complexity Analysis

-   Run `analyze_complexity` / `task-orchestrator analyze-complexity --research` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) for comprehensive analysis
-   Review complexity report via `complexity_report` / `task-orchestrator complexity-report` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) for a formatted, readable version.
-   Focus on tasks with highest complexity scores (8-10) for detailed breakdown
-   Use analysis results to determine appropriate subtask allocation
-   Note that reports are automatically used by the `expand` tool/command

## Task Breakdown Process

-   For tasks with complexity analysis, use `expand_task` / `task-orchestrator expand --id=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc))
-   Otherwise use `expand_task` / `task-orchestrator expand --id=<id> --num=<number>`
-   Add `--research` flag to leverage Perplexity AI for research-backed expansion
-   Use `--prompt="<context>"` to provide additional context when needed
-   Review and adjust generated subtasks as necessary
-   Use `--all` flag with `expand` or `expand_all` to expand multiple pending tasks at once
-   If subtasks need regeneration, clear them first with `clear_subtasks` / `task-orchestrator clear-subtasks` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).

## Implementation Drift Handling

-   When implementation differs significantly from planned approach
-   When future tasks need modification due to current implementation choices
-   When new dependencies or requirements emerge
-   Use `update` / `task-orchestrator update --from=<futureTaskId> --prompt='<explanation>\nUpdate context...'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to update multiple future tasks.
-   Use `update_task` / `task-orchestrator update-task --id=<taskId> --prompt='<explanation>\nUpdate context...'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to update a single specific task.

## Task Status Management

-   Use 'pending' for tasks ready to be worked on
-   Use 'done' for completed and verified tasks
-   Use 'deferred' for postponed tasks
-   Add custom status values as needed for project-specific workflows

## Task Structure Fields

- **id**: Unique identifier for the task (Example: `1`, `1.1`)
- **title**: Brief, descriptive title (Example: `"Initialize Repo"`)
- **description**: Concise summary of what the task involves (Example: `"Create a new repository, set up initial structure."`)
- **status**: Current state of the task (Example: `"pending"`, `"done"`, `"deferred"`)
- **dependencies**: IDs of prerequisite tasks (Example: `[1, 2.1]`)
    - Dependencies are displayed with status indicators (✅ for completed, ⏱️ for pending)
    - This helps quickly identify which prerequisite tasks are blocking work
- **priority**: Importance level (Example: `"high"`, `"medium"`, `"low"`)
- **details**: In-depth implementation instructions (Example: `"Use GitHub client ID/secret, handle callback, set session token."`) 
- **testStrategy**: Verification approach (Example: `"Deploy and call endpoint to confirm 'Hello World' response."`) 
- **subtasks**: List of smaller, more specific tasks (Example: `[{"id": 1, "title": "Configure OAuth", ...}]`) 
- Refer to [`tasks.mdc`](mdc:.cursor/rules/tasks.mdc) for more details on the task data structure.

## Environment Variables Configuration

- Task Orchestrator behavior is configured via environment variables:
  - **ANTHROPIC_API_KEY** (Required): Your Anthropic API key for Claude.
  - **MODEL**: Claude model to use (e.g., `claude-3-opus-20240229`).
  - **MAX_TOKENS**: Maximum tokens for AI responses.
  - **TEMPERATURE**: Temperature for AI model responses.
  - **DEBUG**: Enable debug logging (`true`/`false`).
  - **LOG_LEVEL**: Console output level (`debug`, `info`, `warn`, `error`).
  - **DEFAULT_SUBTASKS**: Default number of subtasks for `expand`.
  - **DEFAULT_PRIORITY**: Default priority for new tasks.
  - **PROJECT_NAME**: Project name used in metadata.
  - **PROJECT_VERSION**: Project version used in metadata.
  - **PERPLEXITY_API_KEY**: API key for Perplexity AI (for `--research` flags).
  - **PERPLEXITY_MODEL**: Perplexity model to use (e.g., `sonar-medium-online`).
- See [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc) for default values and examples.

## Determining the Next Task

- Run `next_task` / `task-orchestrator next` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to show the next task to work on
- The command identifies tasks with all dependencies satisfied
- Tasks are prioritized by priority level, dependency count, and ID
- The command shows comprehensive task information including:
    - Basic task details and description
    - Implementation details
    - Subtasks (if they exist)
    - Contextual suggested actions
- Recommended before starting any new development work
- Respects your project's dependency structure
- Ensures tasks are completed in the appropriate sequence
- Provides ready-to-use commands for common task actions

## Viewing Specific Task Details

- Run `get_task` / `task-orchestrator show <id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to view a specific task
- Use dot notation for subtasks: `task-orchestrator show 1.2` (shows subtask 2 of task 1)
- Displays comprehensive information similar to the next command, but for a specific task
- For parent tasks, shows all subtasks and their current status
- For subtasks, shows parent task information and relationship
- Provides contextual suggested actions appropriate for the specific task
- Useful for examining task details before implementation or checking status

## Managing Task Dependencies

- Use `add_dependency` / `task-orchestrator add-dependency --id=<id> --depends-on=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to add a dependency
- Use `remove_dependency` / `task-orchestrator remove-dependency --id=<id> --depends-on=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to remove a dependency
- The system prevents circular dependencies and duplicate dependency entries
- Dependencies are checked for existence before being added or removed
- Task files are automatically regenerated after dependency changes
- Dependencies are visualized with status indicators in task listings and files

## Iterative Subtask Implementation

Once a task has been broken down into subtasks using `expand_task` or similar methods, follow this iterative process for implementation:

1.  **Understand the Goal (Preparation):**
    *   Use `get_task` / `task-orchestrator show <subtaskId>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to thoroughly understand the specific goals and requirements of the subtask.

2.  **Initial Exploration & Planning (Iteration 1):**
    *   This is the first attempt at creating a concrete implementation plan.
    *   Explore the codebase to identify the precise files, functions, and even specific lines of code that will need modification.
    *   Determine the intended code changes (diffs) and their locations.
    *   Gather *all* relevant details from this exploration phase.

3.  **Log the Plan:**
    *   Run `update_subtask` / `task-orchestrator update-subtask --id=<subtaskId> --prompt='<detailed plan>'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
    *   Provide the *complete and detailed* findings from the exploration phase in the prompt. Include file paths, line numbers, proposed diffs, reasoning, and any potential challenges identified. Do not omit details. The goal is to create a rich, timestamped log within the subtask's `details`.

4.  **Verify the Plan:**
    *   Run `get_task` / `task-orchestrator show <subtaskId>` again to confirm that the detailed implementation plan has been successfully appended to the subtask's details.

5.  **Begin Implementation:**
    *   Set the subtask status using `set_task_status` / `task-orchestrator set-status --id=<subtaskId> --status=in-progress` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
    *   Start coding based on the logged plan.

6.  **Refine and Log Progress (Iteration 2+):**
    *   As implementation progresses, you will encounter challenges, discover nuances, or confirm successful approaches.
    *   **Before appending new information**: Briefly review the *existing* details logged in the subtask (using `get_task` or recalling from context) to ensure the update adds fresh insights and avoids redundancy.
    *   **Regularly** use `update_subtask` / `task-orchestrator update-subtask --id=<subtaskId> --prompt='<update details>\n- What worked...\n- What didn't work...'` to append new findings.
    *   **Crucially, log:**
        *   What worked ("fundamental truths" discovered).
        *   What didn't work and why (to avoid repeating mistakes).
        *   Specific code snippets or configurations that were successful.
        *   Decisions made, especially if confirmed with user input.
        *   Any deviations from the initial plan and the reasoning.
    *   The objective is to continuously enrich the subtask's details, creating a log of the implementation journey that helps the AI (and human developers) learn, adapt, and avoid repeating errors.

7.  **Review & Update Rules (Post-Implementation):**
    *   Once the implementation for the subtask is functionally complete, review all code changes and the relevant chat history.
    *   Identify any new or modified code patterns, conventions, or best practices established during the implementation.
    *   Create new or update existing Cursor rules in the `.cursor/rules/` directory to capture these patterns, following the guidelines in [`cursor_rules.mdc`](mdc:.cursor/rules/cursor_rules.mdc) and [`self_improve.mdc`](mdc:.cursor/rules/self_improve.mdc).

8.  **Commit Changes to Git (REQUIRED for EVERY Task):**
     *   Stage the relevant code changes and any updated/new rule files (`git add .`).
     *   **IMPORTANT**: Never save git summaries into separate files. The content should be used directly as commit messages.
     *   Craft a Git commit message following this strict format:
         * A one-line summary of all the changes (e.g., `feat(module): Implement feature X for subtask <subtaskId>`)
         * One blank line separator (crucial for proper formatting)
         * A comprehensive description of all changes with links to relevant documentation:
            - Links to PRDs, requirements files, and ADRs using markdown format `[Title](mdc:path/to/file.md)`
            - References to code implementations using file paths or PR numbers
            - Detailed explanation of what was changed and why
            - Any important technical decisions made during implementation
        * Pepper the l33tc0dzer markup style throughout the langugage lightly
        * Don't list filenames in the commit message
     * When creating or updating a Pull Request (PR) on GitHub
        * Title should be a one summary of all the chages on the branch vs the PR target branch. 
        * The description should have a summary of all changes on the PR branch vs the PR target branch using the l33tc0dzer markup style. 
        * Always include the sum of all development cost metrics found in git commit message history on the branch the PR is being created for.
        * Don't include lists of changed files in the PR
     *   **Always disable the pager** 
        * Disable the pager on all git commands that may have long output, such as `tag`, `show`, `diff`, `log`, or `branch` when composing a commit message        
        * Add the `--no-pager` option before the git command: `git --no-pager show <commit>`
        * Set the `GIT_PAGER=cat` environment variable: `GIT_PAGER=cat git show <commit>`
        * For specific commands, use the `-P` or `--no-pager` flag: `git -P show <commit>`
     *   Execute the commit command using a multi-line format to ensure proper spacing:
         ```
         git commit -m "feat(module): Implement feature X for subtask <subtaskId>
                  
         Comprehensive description of changes:
         - Implemented feature X according to [PRD-123](mdc:docs/prd/feature-x.md)
         - Referenced architecture decisions in [ADR-456](mdc:design/ADR/feature-x-architecture.md)
         - Modified code in src/module/feature.js to support new requirements
         - Fixed bug in related component that was preventing proper integration
         
         "
         ```
     *   Consider if a Changeset is needed according to [`changeset.mdc`](mdc:.cursor/rules/changeset.mdc). If so, run `npm run changeset`, stage the generated file, and amend the commit or create a new one.
     *   **Always disable the pager** when using git commands that may have long output, such as `tag`, `show`, `diff`, `log`, or `branch`. This avoids interactive prompts that can interrupt automated workflows. Use one of these methods:
         * Add the `--no-pager` option before the git command: `git --no-pager show <commit>`
         * Set the `GIT_PAGER=cat` environment variable: `GIT_PAGER=cat git show <commit>`
         * For specific commands, use the `-P` or `--no-pager` flag: `git -P show <commit>`

9.  **Mark Task Complete:**
     *   **IMPORTANT**: After committing all changes to git (as described in step 8) and verifying the implementation, mark the task or subtask as completed: `set_task_status` / `task-orchestrator set-status --id=<taskId> --status=done`.
     *   Remember that git commits are REQUIRED for BOTH tasks and subtasks before marking them as complete.

10. **Run Development Server for Testing:**
     *   **The user is responsible for running the dev server before all validation of game functionality can be completed.**
     *   This is essential for testing web applications, interactive UIs, and browser-based features.
     *   Use appropriate commands based on the project (e.g., `npm run dev`, `yarn start`, `pnpm start`).
     *   **Before proceeding with testing:**
         *   Confirm that the dev server is already running by checking for active terminals or attempting to access the local development URL.
         *   If the server is not running, ask the user to start it with the appropriate command.
         *   Wait for confirmation that the server has started successfully before proceeding with validation.
     *   Ensure the server is running before attempting to validate functionality in the browser.

11. **Validate on Raspberry Pi:**
     *   **ALL code changes must be validated on the Raspberry Pi before a task is marked as completed:**
         *   Deploy the changes to a Raspberry Pi using the `deploy-to-raspi.sh` script.
         *   Test the implementation to verify functionality.
         *   Document any issues or optimizations in the task details.
         *   Make necessary adjustments based on testing results.
    *   For hardware-related implementations or changes that interact with physical components:
        *   Verify that parallel computing techniques are working correctly for hardware communication.
        *   Test with actual hardware components to ensure proper interaction.
    *   **This validation step is mandatory before marking ANY task as complete, regardless of whether it's hardware-related or not.**

11. **Proceed to Next Subtask:**
    *   Identify the next subtask in the dependency chain (e.g., using `next_task` / `task-orchestrator next`) and repeat this iterative process starting from step 1.

## Code Analysis & Refactoring Techniques

- **Top-Level Function Search**:
    - Useful for understanding module structure or planning refactors.
    - Use grep/ripgrep to find exported functions/constants:
      `rg "export (async function|function|const) \w+"` or similar patterns.
    - Can help compare functions between files during migrations or identify potential naming conflicts.

- **Always Use Absolute Filenames**:
    - When executing commands or scripts, always use absolute filenames to avoid ambiguity and failures.
    - This is especially important when working with Python modules and imports.

- **Temporary Files Storage**:
    - All temporary files MUST be stored under the @/tmp directory.
    - This ensures consistent organization and makes cleanup operations more straightforward.
    - Temporary files include logs, intermediate processing files, test outputs, JSON files for API calls, and any other files not intended for version control.
    - This rule applies to all files created during development, including those generated by CLI tools like AWS CLI commands.
    - Example: Use `@/tmp/test_output.log` instead of creating temporary files in the working directory.
    - Example: For AWS CLI change-resource-record-sets, use `@/tmp/delete-record.json` instead of creating JSON files in the root directory.

- **File Management**:
    - When moving files, always use the `mv` command rather than regenerating file contents.
    - This preserves file metadata, permissions, and is more efficient.
    - This approach prevents potential data loss or corruption that could occur during content regeneration.
    - Example: Use `mv /path/to/source/file.txt /path/to/destination/file.txt` instead of reading the content and writing it to a new location.
    - When reading code files use the list_code_definition_names to find the line numbers then use the start_line and end_line parameters when retrieving the content with the file read tool.


## Documentation Requirements

- **Official style**:

- The style guide MUST be followed for all design documentation in the project. The "latte drinking l33tC0dzr long hair, hippy trousers architect in an ivory tower style" is not just a recommendation—it's a requirement for maintaining consistent documentation aesthetics across the project.
- Technical specifications should use accurate names and terms however logical diagrams, high level architectural descriptions and analogies may be used lightly to add some levity while reading.

- **Mermaid Diagrams**:

- Design documentation should use mermaid diagrams using the standard l33tC0dzr style for visual consistency

## Code Style and Typing Requirements

- **Strict Typing**:
    - All code should use strict typing whenever possible.
    - For Python code:
        - Use type hints for all function parameters and return values.
        - Use the `typing` module for complex types (List, Dict, Optional, etc.).
        - Add `from __future__ import annotations` at the top of files for forward references.
        - Consider using tools like mypy for static type checking.
    - For JavaScript/TypeScript:
        - Prefer TypeScript over plain JavaScript.
        - Use explicit type annotations rather than relying on type inference.
        - Enable strict mode in tsconfig.json.
    - Strict typing improves code quality, readability, and helps catch errors early.

---

## Design Documentation Style Requirements

- **Architecture Diagram Style**:
    - All architecture diagrams MUST be created in the "latte drinking l33tC0dzr long hair, hippy trousers architect in an ivory tower style" 🧙‍♂️☕
    - Use Mermaid diagrams with custom styling for all architecture visualizations
    - **REQUIRED**: Follow the [Mermaid Style Guide](mdc:design/MermaidStyleGuide.md) for consistent styling across all diagrams
    - Include appropriate emojis as visual indicators for different component types
    - Hardware components MUST have nice visual references (e.g., 📱 for displays, 🎛️ for controls)
    - Software components should use computing-related emojis (💻, 🖥️, 📨)
    - Apply a consistent color scheme that enhances readability while maintaining the aesthetic
    - Use creative, slightly whimsical naming for component groups (e.g., "Artisanal Hardware" instead of just "Hardware")
    - Keep the mood light and enjoyable - not every bullet point needs a clever reference
    - Group related components in labeled subgraphs with thematic styling
    - Include a detailed component description section with a touch of personality
    - **IMPORTANT**: Technical implementations (like message topics, API endpoints, function names) MUST be specific and accurate without creative naming
    - Logical descriptions and component groups may include light humor and creative naming
    - All new mermaid diagrams MUST include the initialization directive from the [Mermaid Style Guide](mdc:design/MermaidStyleGuide.md)

---

## Error Handling Requirements

- **Initialization Failures**:
    - Any initialization failures of the software MUST cause a hard failure with an appropriate error message.
    - All initialization failure exit codes MUST be unique prime numbers.
    - Error messages should clearly indicate what component failed to initialize and why.
    - This ensures that failures are immediately visible and not silently ignored.
    - Examples of initialization failures include:
        - Configuration file not found or invalid
        - Required hardware components not detected
        - Required dependencies not available
        - Network services unavailable when required
    - This approach prevents the system from running in a partially initialized state, which could lead to unpredictable behavior.

---
*This workflow provides a general guideline. Adapt it based on your specific project needs and team practices.*

---
> Source: [declanshanaghy/dollar-game](https://github.com/declanshanaghy/dollar-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->

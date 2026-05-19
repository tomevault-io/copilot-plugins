## 020-tasks-workflow

> Task Workflow System for project implementation across multiple task files within the tasks/ directory.


# Task Workflow System (Multi-File in `tasks/` Directory)

- **Purpose:** Standardize task implementation with proper tracking and status updates across a distributed task list.
- **Implementation:** Tasks are distributed across `tasks/tasks-N.md` files, indexed by `tasks/tasks-index.md`.

## Overall Task Management Structure

- **Index File (`tasks/tasks-index.md`):**

  - Contains the **Overall Project Task Summary** (Total Tasks, Pending, Complete for the _entire_ project).
  - Contains the **Task File Index**, mapping Task ID ranges to specific `tasks/tasks-N.md` files.

    ```markdown
    ## Overall Project Task Summary

    - **Total Tasks**: {Total for project}
    - **Pending**: {Pending for project}
    - **Complete**: {Complete for project}

    ## Task File Index

    - `tasks/tasks-1.md`: Contains Tasks {start_id_1} - {end_id_1} ({task_count_1} tasks)
    - `tasks/tasks-2.md`: Contains Tasks {start_id_2} - {end_id_2} ({task_count_2} tasks)
    - ...
    ```

- **Task Files (`tasks/tasks-N.md`):**

  - Each file starts with its own local summary:

    ```markdown
    ## Summary (tasks-N.md) // Clarify which task file this summary is for

    - **Tasks in this file**: {Count for this file}
    - **Task IDs**: {start_id_N} - {end_id_N} // e.g., 001 - 030
    ```

  - Followed by the `## Tasks` section containing individual tasks.

- **Task Structure (within `tasks/tasks-N.md`)**

  - Tasks are defined with:

    ````markdown
    ### Task ID: {ID}

    - **Title**: Example title
    - **File**: example/file/path (relative to project root, excluding project root dir)
    - **Complete**: [ ]

    #### Prompt:

    ```markdown
    DETAILED PROMPT INCLUDING TARGET FILE PATH AND CONTEXT
    ```
    ````

    ```

    ```

## Task Management Workflow

### 1. Locating a Task

- **When user requests "next task":**
  1. Open and consult `tasks/tasks-index.md` to understand the overall project status and identify the sequence of task files.
  2. Iterate through `tasks/tasks-1.md`, `tasks/tasks-2.md`, etc., in numerical order as listed in the index.
  3. Within each `tasks/tasks-N.md` file, find the first task where `Complete: [ ]`. This is the next task.
  4. Note the `tasks/tasks-N.md` file this task resides in.
- **When user specifies a Task ID (e.g., "Task 042"):**
  1. Open and consult `tasks/tasks-index.md`.
  2. Read the "Task File Index" to determine which `tasks/tasks-N.md` file contains that Task ID.
  3. Open the identified `tasks/tasks-N.md` file (e.g., if Task 042 is in `tasks-2.md`, open `tasks/tasks-2.md`).
  4. Locate the task with the specified `### Task ID: {ID}`.

### 2. Task Implementation

- **Always review overall project context as needed:**
  ```javascript
  // ✅ DO: If architectural context is needed, refer to `docs/architecture.md` and other relevant architecture documents located in the `docs/` directory.
  // The task prompt itself should be the primary guide for implementation details.
  ```
- **Implement each task according to its prompt and specified `File` path.**
  ```javascript
  // ✅ DO: Focus implementation *only* on the file specified in the task's "File:" field.
  // ✅ DO: Follow the detailed instructions, context, and references within the task's "Prompt:" section.
  ```

### 3. Updating Task Status (Critical)

- After successfully implementing a task:
  1. **Locate the task** within its specific `tasks/tasks-N.md` file (as identified in Step 1).
  2. **Change its status:** Modify `Complete: [ ]` to `Complete: [x]` in that specific `tasks/tasks-N.md` file.
  3. **Update the local summary within that `tasks/tasks-N.md` file (Optional but Recommended):**
     - If feasible, decrement "Pending" and increment "Complete" counts for _that file's specific summary_. This is secondary to updating the main index.
  4. **Update the Overall Project Task Summary in `tasks/tasks-index.md`:**
     - Decrement "Pending" by 1.
     - Increment "Complete" by 1.
  5. **Save changes** to both the modified `tasks/tasks-N.md` file and `tasks/tasks-index.md`.
  ```javascript
  // ✅ DO: Update "Complete: [ ]" to "Complete: [x]" in the correct `tasks/tasks-N.md` file.
  // ✅ DO: Update "Pending" and "Complete" counts in `tasks/tasks-index.md`.
  // ❌ DON'T: Modify task IDs, titles, or prompts unless explicitly instructed for a correction.
  ```

### 4. Notification

- Inform the user of successful task completion.
- Provide a summary of changes made (files modified).
- State the Task ID and Title of the completed task and which `tasks/tasks-N.md` file it was in.

## Task Creation Guidelines (If AI needs to add/modify tasks)

- **Adding New Tasks Based on User Requests:**

  1. **Determine Target Task File:** The user or AI must decide which `tasks/tasks-N.md` file the new task belongs to.
     - If inserting into an existing sequence, identify the `tasks/tasks-N.md` file.
     - If adding to the very end of the project and the last `tasks-N.md` is full (e.g., based on line count or task count), a new `tasks/tasks-N+1.md` file might need to be conceptually created (the Task Master agent would typically handle new file creation).
  2. **ID Assignment:**
     - **End of a `tasks/tasks-N.md` file:** Use the next sequential number based on the highest ID _globally_ (from `tasks/tasks-index.md` "Total Tasks").
     - **Between Existing Tasks (within a `tasks/tasks-N.md` file):** Use decimal extension format (e.g., 002-1, 002-2).
     ```javascript
     // ✅ DO: Ensure Task ID reflects implementation order.
     // ✅ DO: Use decimal extension for insertions.
     ```
  3. **Structure:** Add the task to the chosen `tasks/tasks-N.md` file using the standard Task Structure.
  4. **Update Summaries:**
     - Update the local summary in the modified `tasks/tasks-N.md` file (increment "Tasks in this file", adjust ID range if necessary).
     - Update the "Total Tasks" and "Pending" counts in `tasks/tasks-index.md`.
     - If a new `tasks/tasks-N+1.md` file was notionally created, update the Task File Index in `tasks/tasks-index.md`.

- **Task Template (for reference when creating new tasks):**

  ````markdown
  ### Task ID: [ID]

  - **Title**: [Concise description]
  - **File**: [Target file path, no project root, e.g., src/components/MyComponent.js]
  - **Complete**: [ ]

  #### Prompt:

  ```markdown
  **Objective:** ...
  **File to Create/Modify:** {e.g., src/components/MyComponent.js}
  **User Story Context:** ...
  **Detailed Instructions:**
  ...
  **Acceptance Criteria (for this task):**
  ...
  ```
  ````

## Navigation and File Management within `tasks/` Directory

- **To find the "next pending task":**
  1. Open `tasks/tasks-index.md`. Review the "Task File Index".
  2. Starting with `tasks/tasks-1.md` (or the first file listed in the index), open it.
  3. Scan for the first task with `Complete: [ ]`. If found, this is the target task and its file.
  4. If not found in the current file, proceed to the next `tasks/tasks-N.md` file listed in the index and repeat step 3.
- **To find a specific Task ID `{ID}`:**
  1. Open `tasks/tasks-index.md`.
  2. Read the "Task File Index" to identify which `tasks/tasks-N.md` file contains Task ID `{ID}`.
  3. Open that specific `tasks/tasks-N.md` file (e.g., if `tasks-index.md` indicates Task 042 is in `tasks/tasks-2.md`, then open `tasks/tasks-2.md`).
  4. Search within that file for `### Task ID: {ID}`.
- **When updating task status:**
  1. Identify the correct `tasks/tasks-N.md` file for the task being updated.
  2. In that specific file, modify `Complete: [ ]` to `Complete: [x]`.
  3. **Critically, open `tasks/tasks-index.md` and update the "Overall Project Task Summary" (Pending and Complete counts).**
  4. Ensure both modified files (the `tasks/tasks-N.md` and `tasks/tasks-index.md`) are saved.

## Important Considerations for the AI IDE Agent

- All task-related files (`tasks-index.md` and `tasks-*.md`) are located within the `tasks/` directory at the root of the project.
- The `tasks/tasks-index.md` file is the **primary source of truth for overall project progress and for locating tasks within the specific `tasks/tasks-N.md` files.**
- Always update `tasks/tasks-index.md`'s summary when a task in any `tasks/tasks-N.md` file is marked complete.
- When asked for the "next task," ensure you are checking files in the correct numerical order as indicated by the `tasks/tasks-index.md`.
- All other project documentation (PRD, Architecture, Stories, etc.) is expected to be in a `docs/` directory at the root of the project.

---
> Source: [aiurda/devcontext](https://github.com/aiurda/devcontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->

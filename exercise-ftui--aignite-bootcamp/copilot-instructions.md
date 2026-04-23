## aignite-bootcamp

> Generate Tasks: A rule for converting Product Requirements Documents (PRDs) into structured, developer-ready task lists with parent tasks, sub-tasks, and relevant file identification.

# Rule: Generating a Task List from a PRD

## Goal

To guide an AI assistant in creating a detailed, step-by-step task list in Markdown format based on an existing Product Requirements Document (PRD). The task list should guide a developer through implementation.

## Output

- **Format:** Markdown
- **Location:** Append referenced PRD with `# Implementation` section

## Process

1.  **Receive PRD Reference:** The user points the AI to a specific PRD file
2.  **Analyze PRD:** The AI reads and analyzes the functional requirements, user stories, and other sections of the specified PRD.
3.  **Review Relevant Code:**: The AI familiarizes itself with relevant code.
4.  **Phase 1: Generate Parent Tasks:** Based on the PRD analysis, generate the main, high-level tasks required to implement the feature and append to the PRD. Use your judgement on how many high-level tasks to use. It's likely to be in the range from 2-10. Present these tasks to the user in the specified format (without sub-tasks yet). Inform the user: "I have generated the high-level tasks based on the PRD. Ready to generate the sub-tasks? Respond with 'Go' to proceed."
5.  **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
6.  **Phase 2: Generate Sub-Tasks:** Once the user confirms, break down each parent task into smaller, actionable sub-tasks necessary to complete the parent task. Ensure sub-tasks logically follow from the parent task and cover the implementation details implied by the PRD.
7.  **Identify Relevant Files:** Based on the tasks and PRD, identify potential files that will need to be created or modified. List these under the `Relevant Files` section, including corresponding test files if applicable.
8.  **Gather Relevant Notes:** Implementation details that are key for the implementation, and will be helpfull to keep in mind for the implementor.
9.  **Generate Final Output:** Combine the parent tasks, sub-tasks, relevant files, and notes into the final Markdown structure.
10.  **Save Task List:** Save the generated document by appending `# Implementation` section to the PRD .md file

## Output Format

The generated task list _must_ follow this structure:

```markdown
## Relevant Files

- `path/to/potential/file1.ts` - Brief description of why this file is relevant (e.g., Contains the main component for this feature).
- `path/to/file1.test.ts` - Unit tests for `file1.ts`.
- `path/to/another/file.tsx` - Brief description (e.g., API route handler for data submission).
- `path/to/another/file.test.tsx` - Unit tests for `another/file.tsx`.
- `lib/utils/helpers.ts` - Brief description (e.g., Utility functions needed for calculations).
- `lib/utils/helpers.test.ts` - Unit tests for `helpers.ts`.

### Notes

- Follow the coding standards defined in `.cursor/rules/general.mdc` and Magicbox rules.
- Brief description of the new/old flow
- Brief description of some the tricky part, and how we should approach it
- Test plan

## Tasks

- [ ] 1.0 Parent Task Title
  - [ ] 1.1 [Sub-task description 1.1]
  - [ ] 1.2 [Sub-task description 1.2]
- [ ] 2.0 Parent Task Title
  - [ ] 2.1 [Sub-task description 2.1]
- [ ] 3.0 Parent Task Title (may not require sub-tasks if purely structural or configuration)
```

## Interaction Model

The process explicitly requires a pause after generating parent tasks to get user confirmation ("Go") before proceeding to generate the detailed sub-tasks. This ensures the high-level plan aligns with user expectations before diving into details.

## Target Audience

Assume the primary reader of the task list is a **junior developer** who will implement the feature.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EXERCISE-FTUI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

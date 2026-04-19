## cursor-base-rules

> Task organization and documentation guidelines


# IMPORTANT: These instructions must be followed when applying this rule

- STOP.  ONLY apply this rule if the glob pattern above matches; if it does not, then you MUST NOT execute ANY of the rules below.

- Apply this rule **if the user's request text includes any of the following terms** (case-insensitive):  
  `task`, `feature`, `implementation`, `complete task`, `task planning`, `task documentation`

- Apply this rule **if the underlying intent of the user's request is to work with task planning or completion**, even if those exact words are not used.  
  For example: creating task documentation, planning features, or completing implementation work.

- ❗ **Do not apply this rule in any other situation unless you are highly confident that the user is asking to perform a matching action. If that happens, stop and ask the user for confirmation before continuing.**

# Task-Based Development Guidelines

These rules apply when working on any task from the specification discussed in `@file ./Instructions.md` and the `spec` directory. They help you maintain clarity, focus, and consistency when implementing features or fixing bugs.

## 🚨 CRITICAL: Test Rule Compliance

**BEFORE writing ANY test code:**
- **ALWAYS read and apply `.cursor/rules/_test-best-practices.mdc`**
- **ALWAYS read and apply `.cursor/rules/_test-mocking-patterns.mdc`**
- **NEVER write test code without these rules being applied**
- **If working in `test/` directories, these rules are MANDATORY**

## General Workflow

- Work on **one task at a time**.
- **BEFORE starting ANY task (including discovery):**
  - **Establish proper environment isolation** - Read and follow `.cursor/rules/_dev-environment-isolation.mdc`
  - Ask any clarifying questions.
  - Thoroughly review the spec and related tests.
  - Look at the first section of each rule whose name starts with "\_persona" in .cursor/rules. If
    any one of them seems right for the task at hand, then start your reply with "(answering as the
    [insert persona name here])" and apply that rule to the current task. In the case of the
    Principal ML Engineer personal, you must also include all of the rules for the Distinguished
    Engineer persona. For all others, do not combine personas. Pick only one.
- Begin by writing or updating tests for the task.
- Only touch code **relevant to your assigned task**.  
  ❗ Do **not** modify or refactor unrelated parts of the codebase.
- Break complex tasks into logical steps.
  - **Pause for confirmation** before starting each new step.

## 🚨 CRITICAL: Environment Isolation Requirement

**BEFORE starting ANY task (including discovery):**
- **Establish proper environment isolation** - Read and follow `.cursor/rules/_dev-environment-isolation.mdc`
- **For Python projects**: ALWAYS activate virtual environment before running any Python commands
- **For other languages**: Follow appropriate isolation patterns for the project type

This ensures consistent, reproducible development environments and prevents dependency conflicts.

## 🚨 CRITICAL: Test Data Creation Requirements

**BEFORE writing ANY test code or fixtures:**
- **ALWAYS use factory_boy factories for test data creation**
- **NEVER create inline dictionaries, objects, or data structures in fixtures**
- **If a factory doesn't exist for needed test data, create one first**
- **Check existing factories in `tests/factories/` before creating new test data**
- **All test data must go through factory_boy - no exceptions**

This ensures consistency, maintainability, and proper validation of all test data.

Use the following formats to ensure structured planning and documentation when creating or completing a task.

## 1. Task Creation Format

Each task should be documented in `spec/task-XXX-name.md` with the following structure:

    ```markdown
    # Task XXX: [Task Name]

    ## Objective
    [Clear statement of what this task aims to achieve]

    ## Requirements
    - [Specific requirement 1]
    - [Specific requirement 2]
    - ...

    ## Assumptions
    - [Assumption 1]
    - [Assumption 2]
    - ...

    ## Dependencies
    - [Dependency 1]
    - [Dependency 2]
    - ...

    ## Implementation Plan
    1. [Step 1]
    2. [Step 2]
    3. ...

    ## Estimated Complexity
    [Low/Medium/High]
    ```

## 2. Discovery Task (001) Format

The first task (discovery) should additionally include the following sections:

    ```markdown
    ## Research Areas
    - [Research topic 1]
    - [Research topic 2]
    - ...

    ## Technical Validation
    - [Technical aspect to validate 1]
    - [Technical aspect to validate 2]
    - ...

    ## Proof of Concept Requirements
    - [POC requirement 1]
    - [POC requirement 2]
    - ...
    ```

## 3. Task Completion Format

When a task is completed, create `spec/task-XXX-completed.md` with:

    ```markdown
    # Task XXX Completed: [Task Name]

    ## Implementation Summary
    [Summary of what was implemented]

    ## Deviations from Plan
    - [Deviation 1]
    - [Deviation 2]
    - ...

    ## Challenges Encountered
    - [Challenge 1]
    - [Challenge 2]
    - ...

    ## Solutions Applied
    - [Solution 1]
    - [Solution 2]
    - ...

    ## Files Modified
    - [File path 1]
    - [File path 2]
    - ...

    ## Commit Message
    [The git commit message used]

    ## Next Steps
    - [Next step 1]
    - [Next step 2]
    - ...
    ```

## 4. Git Commit Format

Use conventional commits format:

    ```
    [type](task-XXX): [brief description]

    [detailed description if needed]
    ```

- `type` is one of `feat`, `fix`, `test`, `docs`, etc.
- Use imperative mood in the description
- Replace `task-XXX` with the relevant task number

## Examples

- **Input:** "How should I format the discovery task?"  
  **Output:** The discovery task should be created as `spec/task-001-discovery.md` with sections for Objective, Requirements, Assumptions, Dependencies, Implementation Plan, Research Areas, Technical Validation, and Proof of Concept Requirements.

- **Input:** "I've completed the authentication task, how do I document it?"  
  **Output:** Create a `spec/task-XXX-completed.md` file with sections for Implementation Summary, Deviations from Plan, Challenges Encountered, Solutions Applied, Files Modified, Commit Message, and Next Steps.

## Definition of Done ✅

1. All the functionality in the task description has been implemented.
2. Tests for new features are written and passing; all existing tests pass too.
3. Code meets style guidelines and best practices.
4. **For test changes: Run `python scripts/validate_test_compliance.py` and ensure no violations.**
5. **For rule changes affecting testing: Run `python scripts/validate_test_compliance.py` and ensure no violations.**

Once complete, make a Git commit for the change. You may `git init` a local repo if one does not already exist.

## Metadata 🗒️

- Version: 1.1
- Intended use: Standardize task planning, documentation, and closure with enhanced test validation requirements
- Author: Jake Reichert
- Last Updated: 2025-01-20

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jreichert) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

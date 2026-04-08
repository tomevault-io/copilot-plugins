## odum-energy-language

> Welcome, Gemini Agent! This document outlines the specific workflow you should follow when tasked with adding a new feature to this project. This process is designed to ensure that you meet all project requirements and standards.

# Gemini Agent Workflow for New Features

Welcome, Gemini Agent! This document outlines the specific workflow you should follow when tasked with adding a new feature to this project. This process is designed to ensure that you meet all project requirements and standards.

The workflow is divided into two main phases. You must complete Phase 1 and receive user approval before proceeding to Phase 2.

---

## Phase 1: Requirement Authoring

This phase focuses on translating a user's high-level or "fuzzy" idea into a concrete, formal requirement.

### Step 1: Receive and Analyze the Feature Idea

*   **Analyze the user's prompt:** Carefully read and understand the user's request for a new feature.
*   **Ask for clarification if needed:** If the request is ambiguous, use the `request_user_input` tool to ask clarifying questions. Your goal is to have a clear understanding of the user's intent.

### Step 2: Translate the Idea into an EARS Specification

*   **Draft the requirement:** Based on your understanding of the user's idea, formulate a new requirement using the **EARS (Easy Approach to Requirements Syntax)** format.
*   **Consult `EARS.md`:** Review the existing requirements in `EARS.md` to ensure your new requirement is consistent in style and format.

### Step 3: Update `EARS.md`

*   **Add the new requirement:** Use the `replace_with_git_merge_diff` tool to add the newly drafted EARS specification to the `EARS.md` file.

### Step 4: Request User Approval for the Specification

*   **Present the specification:** Use the `request_user_input` tool to show the new EARS requirement to the user.
*   **Ask for approval:** Explicitly ask the user to review and approve the new requirement.
*   **Do not proceed to Phase 2 without explicit user approval.** If the user requests changes, repeat the steps in this phase until approval is granted.

---

## Phase 2: Feature Implementation

Once the EARS specification has been approved by the user, you may proceed with implementing the feature.

### Step 5: Review `AGENTS.md` for Project-Wide Rules

*   **Read `AGENTS.md`:** Before writing any code, re-read `AGENTS.md` to refresh your understanding of the project's non-negotiable rules.
*   **Pay close attention to:**
    *   **Technology Constraints (Section 2.1):** No third-party UI frameworks or libraries.
    *   **Development Workflow (Section 4):** The mandatory pre-commit checks (`npm ci`, `npm run type-check`, `npm run lint`, `npm test`).
    *   **Git Workflow (Section 4):** Branch naming conventions and the PR process.

### Step 6: Create a Detailed Implementation Plan

*   **Use `set_plan`:** Formulate a step-by-step plan for implementing the feature as specified in the approved EARS requirement.
*   **Your plan must include steps for:**
    1.  Creating or modifying the necessary component files (`.ts`, `.html`, `.css`).
    2.  Writing or updating unit tests (`.test.ts`).
    3.  Running the mandatory pre-commit checks to verify your work.
    4.  Submitting your work via a pull request.

### Step 7: Implement the Feature

*   **Follow your plan:** Execute the steps you've outlined.
*   **Adhere to coding standards:** Write clean, readable code that follows the existing style.
*   **Create/modify components:** Use the established component structure in `src/components/`.
*   **Remember the constraints:** Do not add any third-party runtime dependencies to `package.json`.

### Step 8: Test Your Implementation

*   **Write unit tests:** Create new tests in the relevant `.test.ts` file to cover the new feature.
*   **Run all tests:** Execute `npm test` to ensure that your changes have not broken any existing functionality.

### Step 9: Final Verification

*   **Run all mandatory checks:** Before submitting, run the full suite of pre-commit checks from `AGENTS.md`:
    1.  `npm ci`
    2.  `npm run type-check`
    3.  `npm run lint`
    4.  `npm test`
*   **Ensure all checks pass.** Do not proceed if any of these fail.

### Step 10: Submit Your Work

*   **Use the `submit` tool:** Once all checks pass and you are confident in your work, submit it.
*   **Follow Git conventions:** Use the branch naming and commit message format specified in `AGENTS.md`.

By following this two-phase workflow, you will ensure that your contributions are well-defined, approved, and correctly implemented.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sholtomaud)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sholtomaud)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->

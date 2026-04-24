## mn-exemplar

> 1) After “Set up development environment & framework”, the **next task MUST be**:

# Planning/Test Culture Overrides (Project Rule)

## Global Planning Requirements
1) After “Set up development environment & framework”, the **next task MUST be**:
   **“Set up the test framework & test environment”**.
   - Pin test runner (e.g., `go test`), structure (`/internal/...`, `/pkg/...`), fixtures folder, golden files, and CI entrypoints.

2) For **every task in the plan that produces code or config**, add two sibling tasks:
   - **Unit Tests:** “Add/extend unit tests for <task-deliverable>”
   - **Integration Tests (if feasible):** “Add/extend integration tests for <task-deliverable>”
   - If integration testing is **not applicable**, add a sub-bullet: “State why integration test is not applicable and what observable we will monitor instead.”

3) Keep tests **in the same PR** as the code unless explicitly labeled `[defer-tests]`.

## Plan Formatting Rules
- Numbered tasks. Each code task is immediately followed by its test tasks.
- Each task includes: **Goal**, **Definition of Done**, **Artifacts**, **Owner(optional)**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmochel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

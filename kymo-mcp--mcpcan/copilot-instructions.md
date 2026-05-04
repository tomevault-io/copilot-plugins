## mcpcan

> 1. **Documentation Rules**:

# Global Development & Documentation Rules

1. **Documentation Rules**:
   - All development process documents, design specifications, task roadmaps, and TODO tracking files **must** be stored in the `.agent/dev-docs/` directory at the project root.
   - Progress queries and task updates **must** take `.agent/dev-docs/03_01_TODO_Tracking.md` as the **single source of truth**.

2. **Language Rules**:
   - **Simplified Chinese**: Used **only** for interactive dialogues between AI and users, and for project documents under the `.agent/dev-docs/` directory.
   - **Full English Programming**: Except for the above cases, all programming activities (including but not limited to **source code, variable naming, function logic, code comments**) **must** be written entirely in English.
   - **Git Commits**: Git commit messages **must** be in English and follow the Conventional Commits specification.

3. **Status Management**:
   - Update the task status in `.agent/dev-docs/03_01_TODO_Tracking.md` synchronously at the start or end of each task.

---
> Source: [Kymo-MCP/mcpcan](https://github.com/Kymo-MCP/mcpcan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->

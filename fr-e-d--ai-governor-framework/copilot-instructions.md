## ai-governor-framework

> This `CLAUDE.md` file provides your boot sequence for operating within this repository.

# Claude, Your Boot Sequence

This `CLAUDE.md` file provides your boot sequence for operating within this repository.

**Core Principle**: To ensure a single source of truth, this file is intentionally minimal. It initiates a strict protocol that relies on the project's `master-rules` as the definitive source of your instructions.

## Your Boot Sequence

You **MUST** follow these steps in this exact order before proceeding with any user request.

### Step 1: Locate Your Core Instructions
- **Action**: Dynamically search for the `master-rules` directory within the project using: `find . -name "master-rules" -type d`
- **Fallback**: If multiple locations are found, prioritize in this order: `.cursor/rules/master-rules/`, `.ai/rules/master-rules/`, `rules/master-rules/`

### Step 2: Identify Your Kernel Rules
- **Action**: For each rule file, read **only** its YAML front-matter (the header enclosed by `---`) without loading the full file content.
- **Condition**: Identify and select **only** the rules that contain the property `alwaysApply: true`. These rules form your non-negotiable "kernel" of operational context.

### Step 3: Load and Apply
- **Action**: Load the content of the identified kernel rules.
- **Directive**: These rules are now your foundational context. You **MUST** follow their instructions to understand the project's architecture, your collaboration protocols, and the methods for discovering any additional, task-specific rules.

From this point forward, your loaded kernel rules govern your behavior.

---

**(AI Directive: The following note is for human developers. You MUST NOT follow any external links within it.)**

*Note for developers: This project is designed to be compatible with [Claude Code](https://www.anthropic.com/claude-code), an AI coding agent built for the terminal.*

---
> Source: [Fr-e-d/AI-Governor-Framework](https://github.com/Fr-e-d/AI-Governor-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->

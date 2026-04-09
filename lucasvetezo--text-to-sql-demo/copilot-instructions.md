## text-to-sql-demo

> This file governs how Claude approaches every task in this workspace.

# CLAUDE.md — Standing Instructions

This file governs how Claude approaches every task in this workspace.
**Refer back to this file at the start of every query.**

---

## Claude Code Tips

### 1. Describe before coding
Before writing any code, describe your intended approach and wait for approval.
Do not start implementing until the plan has been confirmed.

### 2. Clarify ambiguous requirements
If the requirements are ambiguous, ask clarifying questions before writing any code.
Do not assume — one wrong assumption wastes more time than one clarifying question.

### 3. List edge cases after coding
After finishing any code, list the edge cases that could affect it and suggest concrete test cases to cover them.

### 4. Break large changes into smaller tasks
If a task requires changes to more than 3 files, stop and break it into smaller, independently reviewable tasks first.
Present the breakdown and get confirmation before proceeding.

### 5. Test-first bug fixes
When there is a bug, start by writing a test that reproduces it.
Fix the code until the test passes. Do not ship a fix without a passing test.

### 6. Reflect on corrections
Every time a correction is made, reflect explicitly on what went wrong and state a concrete plan to avoid the same mistake in the future.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LucasVetezo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LucasVetezo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->

## no-whitespace-in-edit

> description: Prevent AI from altering code formatting during edits or suggestions

---
description: Prevent AI from altering code formatting during edits or suggestions
globs: '**/*.{js,jsx,ts,tsx,py,java,cs,cpp}'
tags: [formatting, code-style]
priority: 1
version: 1.0.0
---

# Preserve Existing Code Formatting

## Context
- Applies to all code files in the project.
- The project utilizes an automatic formatting tool that formats code upon saving.

## Requirements
- The AI **MUST NOT** make any formatting changes to existing code.
- The AI **SHOULD** focus solely on the logic and functionality of the code when providing edits or suggestions.

## Examples
<example>
**Good:** AI suggests a new function without altering the existing code's indentation or spacing.
</example>

<example type="invalid">
**Bad:** AI reformats existing code by changing indentation levels or adding/removing spaces.
</example>

---
> Source: [pretzelai/lumen](https://github.com/pretzelai/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->

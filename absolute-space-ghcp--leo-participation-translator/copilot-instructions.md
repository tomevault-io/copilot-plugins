## leo-participation-translator

> **This rule is NON-NEGOTIABLE. It applies to every edit, every file, every session.**

# Minimal Edits — Token-Efficient Editing

**This rule is NON-NEGOTIABLE. It applies to every edit, every file, every session.**

---

## Core Rule: Surgical Edits Only

When making changes to any file — documents, code, configuration, markdown — **never reissue the entire file**. Make only the targeted edit and present the change in context.

### What This Means

- **Documents:** Edit the specific paragraph, section, or table row. Do NOT regenerate the full document.
- **Code:** Use StrReplace to change only the affected lines. Do NOT rewrite entire functions or files.
- **Plans:** Update only the changed section. Do NOT recreate the plan.
- **Presentations to the user:** Show only the diff or changed section with enough surrounding context (3-5 lines) for the user to understand placement.

### Anti-Patterns (NEVER DO)

- Rewriting an entire 200-line file to change 3 lines
- Regenerating a full markdown document to update one table row
- Presenting the complete output of a file after a small edit
- Using the Write tool to overwrite a file when StrReplace would suffice

### Correct Pattern

1. Identify the minimal change needed
2. Use StrReplace (for code/docs) or targeted edit tools
3. Present only the changed portion to the user with brief context
4. If the user needs full context, they will ask for it

---

*This rule saves tokens, reduces errors from unintended changes, and keeps conversations focused.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Absolute-Space-GHCP) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->

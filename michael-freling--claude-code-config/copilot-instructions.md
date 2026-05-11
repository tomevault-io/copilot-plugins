## claude-code-config

> At the beginning of a session or when context needs refreshing, you must perform the **Rule Indexing** phase:

# Gemini CLI Context

## Initialization & Indexing Protocol (Startup)

At the beginning of a session or when context needs refreshing, you must perform the **Rule Indexing** phase:

1.  **Scan Headers**: Run `head -n 5 rules/*.md` (and `~/.claude/rules/*.md` if available) to inspect the YAML frontmatter of all rule files.
2.  **Categorize Rules**:
    *   **Global Rules**: Files *without* a `paths:` property.
        *   **Action**: Read these files immediately and add them to your persistent memory/context.
    *   **Scoped Rules**: Files *with* a `paths:` property (e.g., `paths: "**/*.go"`).
        *   **Action**: Note the filename and the pattern, but **DO NOT** read the full content yet. Keep this map in your working memory.

## Runtime Rule Application

When you are about to read, write, or modify a file:

1.  **Check Index**: Consult your memory of the **Scoped Rules**.
2.  **Match**: If the target file matches a remembered pattern, `read_file` the corresponding rule file *before* proceeding with the action.
3.  **Apply**: specific guidelines from that rule file to your current task.

---
*Note: This file provides the grounding context for the Gemini CLI agent.*

---
> Source: [michael-freling/claude-code-config](https://github.com/michael-freling/claude-code-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->

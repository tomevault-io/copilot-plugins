## zk-agent

> This document outlines the conventions and best practices for Gemini CLI when interacting with Markdown (`.md`) files within this project. Adhering to these guidelines ensures consistency, clarity, and maintainability of documentation.

# Gemini CLI Markdown Interaction Guidelines

This document outlines the conventions and best practices for Gemini CLI when interacting with Markdown (`.md`) files within this project. Adhering to these guidelines ensures consistency, clarity, and maintainability of documentation.

## Core Principles

*   **Respect Existing Structure:** Always analyze the existing Markdown file's structure, headings, lists, code blocks, and formatting before making any modifications. Mimic the established style.
*   **Content Integrity:** Do not alter the meaning or intent of existing content unless explicitly instructed. Focus on adding, updating, or correcting information as per the user's request.
*   **Conciseness and Clarity:** When adding new content, ensure it is concise, clear, and directly addresses the topic. Avoid verbose language or unnecessary details.
*   **Code Block Handling:** When inserting or modifying code blocks, ensure proper syntax highlighting (if applicable) and correct indentation. Do not introduce extraneous characters or formatting.
*   **No Conversational Filler:** Markdown files are for documentation. Do not include conversational elements, agent-specific commentary, or interaction logs within the `.md` file content itself.
*   **Tool Usage:** Utilize `read_file` to understand content, `write_file` for creating new files or overwriting content (with caution), and `replace` for precise modifications. Always verify the exact `old_string` for `replace` operations.
*   **User Intent:** Prioritize understanding the user's specific intent for Markdown file modifications. If ambiguous, seek clarification.

## Specific Actions

*   **Adding Sections:** When adding new sections, ensure they logically fit within the document's flow and hierarchy. Use appropriate heading levels.
*   **Updating Information:** When updating existing information, ensure the changes are accurate and reflect the latest state.
*   **Formatting:** Maintain consistent use of bold, italics, code snippets, lists, and links as per the file's existing conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wilson323)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wilson323)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

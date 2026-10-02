## career-pivot

> When providing code changes:

# OSai Project Agent Rules

## Non-Negotiable Coding Rules

When providing code changes:

1. NEVER provide partial code snippets for an existing file.

2. If modifying an existing file:
   - Provide the complete file path.
   - Provide the entire replacement file contents from first line to last line.
   - Do not say "add this", "replace this section", "insert this", or "update this function".

3. If a file is too large for one response:
   - State that the file is too large.
   - Split the complete file into clearly labeled parts.
   - Do not summarize or omit sections.

4. Before replacing a file with a significantly different version:
   - Alert the user.
   - Explain what is changing.
   - Explain what may be removed.
   - Wait for approval before providing the replacement.

5. Never reduce documentation files by removing existing architecture decisions unless the user explicitly requests it.

6. Treat documentation as cumulative history:
   - Append updates when possible.
   - Mark deprecated sections instead of deleting them.

7. Every coding response should include:
   - Files changed
   - Complete replacement code
   - Testing step
   - Git commit command

## Architecture Rules

1. The shell owns layout and placement.

2. Modules own content, data, views, and intelligence.

3. The shell should not hardcode module-specific business logic.

4. Modules live in:

```text
modules/

---
> Source: [OrbitSystemsAi/career-pivot](https://github.com/OrbitSystemsAi/career-pivot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->

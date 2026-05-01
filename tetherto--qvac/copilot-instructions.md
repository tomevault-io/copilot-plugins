## file-formatting

> File formatting rules for consistent code style


# File Formatting Rules

## Always Add Newline at End of Files

**CRITICAL**: ALWAYS ensure that EVERY file ends with a newline character (`\n`).

### Rules

- **ALWAYS add a newline at the end of EVERY file** you create or edit
- This applies to ALL file types without exception:
  - All code files (any programming language)
  - All configuration files (.json, .yml, .yaml, .toml, etc.)
  - All documentation files (.md, .txt, etc.)
  - All script files (.sh, .bat, etc.)
  - Basically EVERY text-based file

### Why This Matters

- Prevents "No newline at end of file" warnings in git diffs (the red circle)
- Follows POSIX standard for text files
- Ensures clean git history and proper file processing
- Many tools expect and require files to end with newlines

### Implementation

When using Write, Edit, or MultiEdit tools:
- **ALWAYS** ensure the content ends with `\n`
- For multi-line content, add newline after the last line
- For single-line content, still add the newline
- This is NOT optional - it's required for every file

### Examples

✅ **Correct** - All files end with newline:

```javascript
const foo = "bar";
module.exports = foo;

```

```json
{
  "name": "example",
  "version": "1.0.0"
}

```

❌ **Wrong** - Missing newline at end:

```javascript
const foo = "bar";
module.exports = foo;```

```json
{
  "name": "example",
  "version": "1.0.0"
}```

---
> Source: [tetherto/qvac](https://github.com/tetherto/qvac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->

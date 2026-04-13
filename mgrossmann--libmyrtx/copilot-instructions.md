## libmyrtx

> | Rule | Pattern | Description |

# Cursor Rules for libmyrtx

## Code Style & Standards

| Rule | Pattern | Description |
|------|---------|-------------|
| Enforce C99 Standards | `\.c$\|\.h$` | Ensure all C code adheres to C99 standards |
| Header Guard Convention | `\.h$` | All headers should use MYRTX_MODULE_FILENAME_H guard pattern |
| Documentation Required | `\.c$\|\.h$` | All public functions must have documentation comments |
| Memory Management | `\.c$` | Use arena allocator where possible, avoid raw malloc/free |
| Error Handling | `\.c$` | Use the context system for error propagation |
| Naming Convention | `\.c$\|\.h$` | Use myrtx_module_function naming pattern for all public functions |
| Test Coverage | `src/.*\.c$` | Each source file should have corresponding tests |
| Language | `\.c$\|\.h$\|\.md$\|\.rst$` | All comments and documentation must be written in English, even though team communication is in German |
| Indentation | `\.c$\|\.h$` | Use tabs with a tab width of 4 spaces |
| Documentation Maintenance | `\.rst$` | Documentation files must always be kept up-to-date with code changes |

## Editor Settings

- **Format on Save**: Enabled
- **Auto-fix on Save**: whitespace, semicolon
- **Lint on Save**: Enabled
- **Tab Size**: 4
- **Insert Type**: Tab (not spaces) 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgrossmann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

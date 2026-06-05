## jetson

> BEGIN EVERY TASK BY THE STANDARDS SET FORTH IN THIS FILE.

# .cursorrules

BEGIN EVERY TASK BY THE STANDARDS SET FORTH IN THIS FILE.

## PROHIBITED ACTIONS

- Solutions based solely on existing code examples
- Skipping documentation review even when similar patterns exist elsewhere
- Assuming pattern consistency without verification
- Adding wrapper functions around existing methods
- Altering existing functionality
- Creating your own implementations of existing methods
- Making assumptions about the code - always verify first
- Creating actual instances of real classes, instead use the existing methods directly
- Creating new files when asked to correct a file

## CODEBASE CONTEXT SOURCE

- Use the `.docs/changelog/00_INDEX.md` file to navigate the changelog.
- `MASTER_CODEBASE_ARCHIVE.md` is the primary source of truth for code patterns, structures, interfaces, and relationships. Always refer to this file first when seeking context for code generation, correction, or analysis.
- Maintain a new `CODEBASE_ARCHIVE.md` file in the `.docs/changelog` directory.
- Use the `CODEBASE_ARCHIVE.md` file to document and track codebase changes and code implementations.
- Format new context entries according to the established structure
- Ensure new entries include concrete code examples, not explanations

## ERROR CORRECTION STANDARDS

`.CURSOR_ERROR_CORRECTION.md` is the primary source of truth for error correction standards. Abide by this file when correcting errors.

## CODE CHANGES PROCESS

1. Review the Codebase Architecture file `.docs/changelog/MASTER_CODEBASE_ARCHIVE.md` to ensure relevant system patterns and implementations are used.
2. Search `.docs/changelog/CodeBase_Modules` for specific system patterns and implementations not found in `MASTER_CODEBASE_ARCHIVE.md`. Verify implementation approaches against established patterns before generating code.
3. Generate complete and fully functional code blocks. If the generated code is too larage for one response, break the generation up into steps.
4. Document the changes in the `CODEBASE_ARCHIVE.md` file.

## TASKLIST PLANNING FORMAT

Maintain a Tasklist in `.CURSOR_SCRATCHPAD.md` section below of actionable steps in this format:

```
{current_date}
- [ ] Task
  - [ ] Subtask
```

---
> Source: [deadcoast/JETSON](https://github.com/deadcoast/JETSON) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->

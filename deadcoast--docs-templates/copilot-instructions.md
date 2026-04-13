## docs-templates

> - **`Cursor System Documents`**: *Optimized Document Format for AI*

# CURSORRULES

## `System Reference Files` (`SRF`)
- **`Cursor System Documents`**: *Optimized Document Format for AI*
  - `CodeBase_Docs/Cursor_System_Architecture.md` - Phased restructuring plan with implementation priorities
  - `CodeBase_Docs/Cursor_System_Integration.md` - Current state architecture reference
- **`CodeBase Context Documents`**: *Context Documents on prior Code Implementation*
  - `CodeBase_Docs/System_Directory_Tree.md` - An outline of the entire CodeBase for contextual awareness
  - `CodeBase_Docs/System_Development_History.md` - Comprehensive Documentation on the History of Development on each section of the code base.

## Workflow & Instructions
1. **Context First**: Always check `System_Directory_Tree.md` and relevant `SRF` before generating code. If you cannot find context still, provide a grep or command search on the `CodeBase_Context.md` file for relevant implementations.
2. **Architecture Alignment**: Ensure all implementations align with specifications in `SRF` Documents. If you still cannot find proper context or current implementations, **ask the user** before generating a new implementation.
3. **Incremental Implementation**: Generate code in manageable steps, maintaining consistency with existing codebase.
4. **Knowledge Management**: Document reusable components, fixes, and implementation decisions in the appropriate `SRF`.

5. **Rule of Seven (`RO7`)**:
   1. Review `Scratchpad` for current task and plan implementation steps
   2. Search `SRF` for existing implementations; adjust plan accordingly
   3. Take action to complete the task
   4. Review for missing implementations required by recent changes
   5. Identify implementation gaps and plan necessary steps
   6. Update Cursor System documents `Cursor_System_Architecture.md` and `Cursor_System_Integration` to maintain codebase consistency
   7. Update the `Scratchpad`, with progress


6. **`Scratchpad` Usage**: Maintain a task list in this file following this format:
   ```
   ## Scratchpad
   
   [X] Task 1
   [ ] Task 2
   ```
   
   Update progress markers as subtasks are completed. Use the Scratchpad to maintain implementation context and track progress.

---

# Scratchpad

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deadcoast) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

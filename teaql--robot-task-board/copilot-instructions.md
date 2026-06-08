## robot-task-board

> When modifying or generating code for this repository, you MUST adhere to the following rules:

# AI Assistant Rules for TeaQL Robot Task Board

When modifying or generating code for this repository, you MUST adhere to the following rules:

## 1. Do NOT Use Repository API for Domain Entities
- **Never** use `save_entity_graph_from()`, `save_entity_with_comment()`, or any other repository-level methods directly on a `Repository` or `ResolvedRepository` instance to persist entities.
- **Why?** Saving via the repository graph API (e.g., `save_entity_graph_from`) executes a full entity graph replacement. If child relations (like execution logs) are not explicitly loaded and re-attached to the graph, the framework will implicitly delete the missing records in the database.

## 2. Use the Entity API (NTTAPI) Exclusively
- Always perform state transitions or updates directly on the domain entity object itself, and call the generated `.save(&ctx).await` method on the entity.
- Example:
  ```rust
  let mut task = ...;
  task.set_comment("Update task status");
  task.save(&ctx).await?; // CORRECT
  ```
- This ensures only the requested changes are applied safely without accidentally deleting unselected child relations.

## 3. Explicit Deletion Only
- Data MUST NOT be deleted implicitly. Unless there is an explicit deletion command or `.mark_as_delete()` is explicitly called on an entity, no command should delete any data.

---
> Source: [teaql/robot-task-board](https://github.com/teaql/robot-task-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->

## task-master

> - **All task-master files must be stored in `.task-master/`**

- **All task-master files must be stored in `.task-master/`**
  - Includes `tasks.json`, `PRD.txt`, and all individual task files (e.g., `.task-master/tasks/1.txt`)
  - Do **not** store these files in `scripts/` or the project root
  - Example structure:
    ```
    .task-master/
      tasks.json
      PRD.txt
      tasks/
        1.txt
        2.txt
      dev.mjs
    scripts/
      example_prd.txt
      README.template.md
    ```

- **The dev script file must be at `.task-master/dev.mjs`**
  - Do **not** place `dev.mjs` or `dev.js` in the `scripts/` folder
  - All task-master CLI commands should reference `.task-master/dev.mjs`
  - Example usage:
    ```sh
    node .task-master/dev.mjs list
    node .task-master/dev.mjs parse-prd --input=.task-master/PRD.txt
    ```

- **Scripts and templates must be stored in `scripts/`**
  - Example: `scripts/example_prd.txt`, `scripts/README.template.md`
  - Do **not** store templates or example PRDs in `.task-master/`

- **File naming conventions**
  - Main PRD file: `.task-master/PRD.txt`
  - Main tasks file: `.task-master/tasks.json`
  - Individual task files: `.task-master/tasks/<id>.txt` (where `<id>` is the task ID)
  - Dev script: `.task-master/dev.mjs`
  - Example/template files: `scripts/`
  - Do **not** use alternative extensions or locations for these files

- **Best Practices**
  - Keep `.task-master/` as the single source of truth for all task-master data and scripts
  - Use `scripts/` only for reference/example/template files
  - Reference this rule in onboarding and documentation for new contributors

- **Anti-patterns**
  ```
  // ❌ DON'T: Place tasks.json or PRD.txt in scripts/
  scripts/tasks.json
  scripts/PRD.txt

  // ❌ DON'T: Place dev.mjs in scripts/
  scripts/dev.mjs

  // ❌ DON'T: Store templates in .task-master/
  .task-master/example_prd.txt
  ```

- **References**
  - See [cursor_rules.mdc](mdc:.cursor/rules/cursor_rules.mdc) for rule formatting
  - Example PRD: [scripts/example_prd.txt](mdc:scripts/example_prd.txt)
  - Example dev script: [.task-master/dev.mjs](mdc:.task-master/dev.mjs)

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->

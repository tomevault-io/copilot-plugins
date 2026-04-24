## speech4

> - Always read `plan.md`** at the beginning of a new conversation to understand the architecture, goals, style and constraints of the project.

### 🔄 Project knowledge and context
- Always read `plan.md`** at the beginning of a new conversation to understand the architecture, goals, style and constraints of the project.
- Check `tasks.md`** before starting a new task. If the task is not listed, add it with a brief description and today's date.
- Use consistent naming conventions, file structures and architectural patterns** as described in `plan.md`.
- When decisions are made, they must be recorded in `plan.md` or `tasks.md` so that the code always matches our plan

### 🧱 Code structure & modularity
- **Never create a file that is longer than 500 lines of code.** If a file approaches this limit, refactor it by splitting it into modules or auxiliary files.
- **Organize code into clearly separated modules**, grouped by function or responsibility.
- **Use clear, consistent imports** (prefer relative imports within packages).

### ✅ Task completion
- Mark completed tasks in `tasks.md`** immediately after their completion.
- Add new subtasks or TODOs discovered during development to the current feature in `tasks.md` under a "Discovered During Work" section.

### Commenting out code

It is better to delete code than to comment it out, this is only allowed for testing purposes

### Make changes directly instead of asking

For changes that only affect one file and not several, apply them immediately and do not ask for permission if there are no options for me to choose from.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoschi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->

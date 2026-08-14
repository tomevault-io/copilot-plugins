## sudokusensei

> - When starting or restarting the project, stop the previously recorded SudokuSensei backend and frontend process trees first.

# Development workflow

- When starting or restarting the project, stop the previously recorded SudokuSensei backend and frontend process trees first.
- Use the root `start-dev.bat` launcher so process IDs are tracked and duplicate development servers are not left running.
- Use `stop-dev.bat` when the task is finished or when the user asks to stop the project.
- Never terminate unrelated Node.js or Python processes; only stop processes recorded by this project's launcher.

---
> Source: [IbukunSanni/SudokuSensei](https://github.com/IbukunSanni/SudokuSensei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->

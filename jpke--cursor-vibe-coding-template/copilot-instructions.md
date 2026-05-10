## development

> Core development practices for frontend, testing, and Python environments


## Frontend Development
- **MVP Development Workflow**:
  - **Start with HTML**: Every application MVP, regardless of its final platform (e.g., mobile app), should begin as a basic HTML page. This enables rapid prototyping and iteration.
  - **Styling Guidelines**: If you provide a `design.json` file, I will use it as the primary guideline for all styling and visual design choices.

## Testing Strategy
- **Test-Driven Development**: Adopt a TDD approach where possible. At a minimum, ensure that unit tests are created or updated for any new or modified code.
- **Test Completion**: All tests must be passing before a coding task is considered complete.
- **Test Output**: All test reports, artifacts, and error logs must be output to a common `tests_output/` directory at the project root.

## Python Environment Management
- **Virtual Environments**: Always use a virtual environment (`venv`) when running Python scripts. Check for an existing `venv` in the current or parent directories before creating a new one. Activate the `venv` before installing packages or executing scripts.

---
> Source: [jpke/cursor-vibe-coding-template](https://github.com/jpke/cursor-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->

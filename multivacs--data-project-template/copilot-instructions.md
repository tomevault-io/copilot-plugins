## data-project-template

> - Use PEP8 guideline for python code

## Conventions
- Use PEP8 guideline for python code
- Use SOLID principles when possible
- Use commit messages conventions

## Organization
- Small functions with only one responsability
- The source code must be located under `src/` folder and organized by data steps: `dataset`, `models/`, `plots`...

## Testing
- Check CI workflows under `.github/workflows`
- Tests are under `tests/` folder
- Execute tests with `pytest test/`
- Not accept code with errors, lint or failing tests
- Add or update tests when new feature or behavior is added/updated

## Commits and Pull Requests
- Title of PR: [<project_name>] Clear description.
- Small PRs.
- Commit messages follows commit conventions with labels: feat, docs, refactor, fix, test, ci, revert
- E.g. commit: `<type>(<optional_subtype>): short description and purpose of change`
- Explain what has changed, why and how has been verified

## Agent behavior
- If a request is not clear, make questions before execution
- Simple tasks and well defined executed directly
- Complex changes (refactors, new features) need confirmation of understanding before execution
- Not make 
- Si una petición no está clara, hacer preguntas concretas antes de ejecutar.
- Tareas simples y bien definidas se ejecutan directamente.
- Cambios complejos (refactors, nuevas features, decisiones de arquitectura) requieren confirmar entendimiento antes de actuar.
- Do not assume implicit requirements. If information is missing, ask for it.

---
> Source: [multivacs/data-project-template](https://github.com/multivacs/data-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
